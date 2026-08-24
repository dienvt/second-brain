---
tags: [comvex, digima, backend, domain, email, migration, microservice, architecture]
project: digima-backend-app, digima-backend-email-api
source: app/Jobs/Email/, app/Services/Email/, app/Services/EmailClient/, app/Http/Controllers/User/v2/Email/, digima-backend-email-proto
created: 2026-08-19
---

# Migration to email-api — Overview

Plan for moving the **CRM email domain** out of the `digima-backend-app` Laravel monolith into the existing `digima-backend-email-api` Go microservice, without interrupting users.

Companions:
- [[Migration to email-api - Waves]] — the ordered, step-by-step plan
- [[Email Feature Usage Metering]] — the billing-critical subsystem, and why it gates the tracking wave
- Current state: [[Relationship Diagram]], [[Email Tracking - Delivery, Open, Click]], [[email]]
- Target service: [[digima-backend-email/api|email-api project note]], [[digima-backend-email/worker|email-api worker note]]

> Mirror of the repo docs at `docs/migrations/email-domain-to-email-api/` in `digima-backend-app`.

## TL;DR

| Question | Answer |
|---|---|
| **Target** | Extend the **existing** `digima-backend-email-api`, not a new service. It already owns inbox emails, envelopes, attachments, links, clicks, opens, connections, providers — and per-account DB provisioning. |
| **Scope** | 8 subsystems: templates, sender identity, composition/send, bulk orchestration, tracking/engagement, attachments, statistics, **feature usage metering** |
| **Out of scope** | All transactional/system mail (password reset, activation, notifications, digests) — separate `MAIL_MAILER` path, no shared tables |
| **How users keep working** | Monolith stays the REST façade; per-account feature flags; dual-write → backfill → verify → read cutover → stop-write → drop; Kafka events replace cross-DB SQL joins |
| **Size** | 7 waves (0–6). Waves 1–3 parallelisable; 4 → 5 → 6 strictly sequential |
| **Biggest blocker** | Who resolves recipients and renders placeholders (§5.1) — decides every proto |
| **Biggest surprise** | email-api has **no SendGrid support at all**. Wave 2 adds it as a new transport, not a port. |
| **Riskiest wave** | Wave 5 (bulk) — a bug means duplicate mail to customers' contact lists |
| **Most constrained** | Wave 3 — it moves the table that **invoices are computed from** |

---

## 1. Why extend email-api rather than build new

`digima-backend-email-api` already exists and is already wired into the monolith:

- **Composer:** `comvex-jp/digima-backend-email-proto ^1.12`
- **Transport:** `app/Services/EmailClient/Client.php` — one gRPC channel, 11 typed service clients, keepalive tuned for the LB, registered in `Providers/GrpcClientServiceProvider.php`
- **Adapter layer:** `app/Services/EmailClient/Handlers/*` extending `BaseHandler` — validates the `email` feature subscription and account feature flag, encodes identity as `Authorization: Bearer base64(json({account_id, account_code, user…}))`, maps gRPC status codes to Digima exceptions
- **Tenancy:** `Handlers/Databases.php` (`Create`/`Delete`) provisions the per-account DB inside email-api; wired to account lifecycle by `Listeners/Email/InitializeEmailDatabaseListener`. email-api is **already database-per-account**, same as the monolith
- **Live traffic:** `/user/v3/emails/*` (inbox, connections, providers, attachments, authorizations, envelopes) already resolves against email-api

Reusing this cuts the work roughly in half: auth, tenancy, the envelope model, attachment storage and the tracking primitives already exist.

### Existing services in email-proto

Attachments, Authorizations, Clicks, Connections, Databases, Emails, Envelopes, FeatureUsage, Links, Opens, Providers.
Existing events: attachment created / bulk deleted, click created, envelope created / bulk deleted.

### Contract gaps — no service today for

Templates · sender Addresses · sender Domains · Exclusion list · envelope Recipients · Recipient events · Unsubscribes · Campaign/bulk orchestration · Statistics

### ⚠️ The SendGrid gap

**email-api knows nothing about SendGrid.** Its `Provider` message is purely SMTP/IMAP server settings (`smtp_config`, `imap_config`), credentials in HashiCorp Vault, sending through *users' own SMTP servers*. There is not one mention of SendGrid anywhere in `digima-backend-email-proto`.

The monolith's CRM email path is entirely SendGrid: subusers, API keys, domain authentication, the event webhook, suppression/unbounce, dedicated vs "dirty" IP pools.

→ Wave 2 is not a port. It **introduces SendGrid as a first-class transport inside email-api**, alongside the existing SMTP/IMAP providers. Budget for it accordingly.

---

## 2. Scope

### In scope

| Subsystem | Monolith surface |
|---|---|
| **A. Templates** | Account templates, folders, labels, placeholders; core (Digima) templates + admin/internal CRUD |
| **B. Sender identity** | `email_addresses`, `email_domains`, SendGrid domain authentication, subuser + API-key management, invalid-domain credits |
| **C. Composition & send** | `Email` model lifecycle, recipient resolution, body/link parsing, placeholder rendering, envelope creation, SendGrid dispatch |
| **D. Bulk orchestration** | Chunked fan-out, concurrency/interval settings, progress tracking, scheduled sending, cancellation |
| **E. Tracking & engagement** | `email_links`, guest click/open/unsubscribe endpoints, SendGrid event webhook ingestion, envelope recipients + events, exclusion list, bounce handling |
| **F. Attachments** | `email_files` pivot, storage usage/overcharge |
| **G. Statistics** | Per-email and account-level envelope statistics |
| **H. Feature usage metering** | Email usage metering for billing, storage metering of email rows and attachments, SendGrid invalid-domain credits — see [[Email Feature Usage Metering]] |

### Explicitly out of scope

System and transactional mail — Laravel Notification channel over `MAIL_MAILER`, an entirely separate code path with no shared tables, jobs or services:

- Password reset (`PasswordReset*Notification`)
- User activation / registration (`UserActivat*Notification`)
- Email address change confirmation (`EmailUpdateConfirmationRequiredNotification`)
- Operational notifications and digests (contact assigned, import/export completed, Andpad/Kintone sync failures, reminders, SMS/LINE/web-form notifications)

> **Boundary case:** `Notifications/Email/*` (`EmailOpenedNotification`, `EmailLinkClickedNotification`, `BouncedRecipientsNotification` + digests) are *system notifications triggered by CRM email events*. The notification stays in the monolith; its **trigger** becomes a Kafka event from email-api.

---

## 3. Current state in the monolith

### 3.1 The send pipeline

Three entry points, one chain:

```
POST /user/v2/emails/{email}/send                  User/v2/Email/EmailsController@send:146
POST /internal/v2|v3/emails/{id}/send-scheduled    Internal/*/EmailsController@sendScheduled:99
Workflow action                                    Services/Workflow/Action/EmailActionExecutor
        │
        ▼
ProcessEmailJob  (dispatch_sync)                   app/Jobs/Email/ProcessEmailJob.php
 ├─ BodyParser::validateEmailLinks / createEmailLinks
 ├─ scheduled?  → markAsScheduled + push to SQS 'scheduler-on-demand'
 │                (scheduler microservice calls back the internal endpoint above,
 │                 substituting the {email_client_code} placeholder in the auth header)
 ├─ classic?    → EnvelopeCreator directly, markAsProcessed
 └─ bulk?       → fire MarkedAsProcessingEvent
                    └─ PrepareBulkEmailContactGroupsSubscriber:54
                         └─ PrepareEmailChunkJob (queued)
                              ├─ counts email_contact_pivot rows (not the contacts table)
                              ├─ reads Setting::KEY_BULK_EMAIL (concurrency 0–500, interval 0–100)
                              ├─ seeds Redis progress counter (1h TTL, matches the 1h SLA)
                              ├─ BulkEmailSentActivityAccumulator::prepare
                              └─ ConcurrentJobDispatcher → N × ProcessEmailChunkJob
                                   (self-chaining by offset/limit, delayed between chunks)
                                   └─ per contact: EnvelopeCreator
                                        └─ CreateEnvelopeJob (dispatch_sync)
                                             └─ DispatchEnvelopeJob
                                                  ├─ decrement invalid-domain credits (prod only)
                                                  └─ Sendgrid\EmailSender
```

> **Completion is inferred, not signalled.** Each `ProcessEmailChunkJob` decrements the Redis counter; the last one to see `<= 0` under `Cache::lock` marks the email processed and fires `ProcessedEvent`. The `failed()` handler continues the chain with a zero processed-count. Wave 5 replaces this with real campaign state.

### 3.2 The feedback loop

```
SendGrid event webhook  POST /sendgrid/v2/events   Sendgrid/v2/EventsController
   └─ ParseRawSendgridEventsJob → StoreSendgridEventsForAccountJob
        └─ email_envelope_recipient_events, email_envelope_recipients
             └─ UpdateStatisticSubscriber, NotifyBouncedRecipientsSubscriber,
                UnBounceEmailAddressListener, LogActivitySubscriber,
                EventProducer\Handlers\RecipientEvents (Kafka)

Guest tracking  GET /email/v2|v3/click, /open · POST /email/v2/unsubscribe
   └─ Email/v2|v3/{Clicks,Opens,Unsubscribes}Controller
        └─ email_envelope_{clicks,opens,unsubscribes} + Web\Tracking\TrackingManager
```

Rate limits to preserve: `throttle.any:20,60` on click, `throttle.email_opens:1,3600` on open.

See [[Email Tracking - Delivery, Open, Click]] for the full semantics, including why opens/clicks are per-envelope but delivery events are per-recipient.

### 3.3 Data

**Per-account DB** (`DatabaseManager::connectToAccountDatabase`) — ~18 tables:

`emails` · `email_contact_pivot` · `email_contact_group_pivot` · `email_addresses` · `email_domains` · `email_envelopes` · `email_envelope_recipients` · `email_envelope_recipient_events` · `email_envelope_clicks` · `email_envelope_opens` · `email_envelope_unsubscribes` · `envelope_statistics` · `email_links` · `email_templates` · `email_template_folders` · `email_template_labels` · `email_template_label_pivot` · `email_files`

**Core DB:** `email_templates`, `email_template_folders`, `email_template_labels` — the Digima-provided templates. Same table names, different connection.

**Statistic DB:** `DatabaseManager::getStatisticConnection()`

See [[Relationship Diagram]] for the ER model.

### 3.4 The twelve consumers outside the email domain

These are the real cost of the migration — each must keep working at every step.

| Consumer | Coupling |
|---|---|
| **Workflow** | `EmailActionExecutor` calls `EnvelopeCreator` in-process; `Workflow\Action.resource_id` references an `Email` row; `CloneWorkflowJob`, `UpdateWorkflowJob`, `DeleteActionJob`, `MarkWorkflowInvalidSubscriber`, `CreateActionRequest` read email/template rows — see [[Workflow]] |
| **Contact filter index** | `FilterClient\Handlers\ContactAllFieldsIndex` ships `email_events` (opened/clicked/bounced/delivered/failed/dispatched/unsubscribed `email_ids`) to filter-api, built from SQL joins across `email_envelope_*` |
| **Contact model** | `Contact::emails()` / `Contact::envelopes()` are `belongsToMany` across `email_contact_pivot` / `email_envelope_recipients`; `getEmailIdsByEvent` and `getEnvelopeEventQuery` join engagement tables directly |
| **Contact export** | `Export\Contact\CreateExportRequestHelper`, `Export\Contact\RowParser` |
| **Activities** | `Listeners/Email/{Bulk,Classic,Inbox}/LogActivitySubscriber`, `Services/Activities/MetaEnricher`, `UpdateActivitiesMetaCommand` |
| **Notifications** | opened / clicked / bounced + digests |
| **Web tracking** | `Services/Web/Tracking/TrackingManager` correlates envelope → website visit |
| **Bulk actions** | `Services/Bulk/Email/BulkActionHelper` (bulk delete of emails) |
| **Kafka producers** | `EventProducer\Handlers\{Emails, EmailEnvelopeClicks, RecipientEvents}` |
| **Feature lifecycle** | `Jobs/Feature/EndSubscriptionJob`, account cancel/disable checks in `ProcessEmailJob` |
| **Feature usage metering** | `Services/Email/UsageManager`, `Services/Storage/UsageManager`, `CalculateStorageOverchargeCommand` — see [[Email Feature Usage Metering]] |
| **Billing** | `Billing\SegmentMaker\UsageBased\EmailSegmentMaker` → `InvoiceCalculator`, `ReconcileUsageService`, `Jobs/Feature/{CalculateFeatureUsageJob,ReconcileDailyUsageJob}` — see [[Feature Usage]], [[Invoice flow]] |

---

## 4. Target state

```
FE ──► monolith REST (/user/v2/emails/*, unchanged shape)
          └─► EmailClient\Handlers\* ──gRPC──► email-api
FE ──► monolith REST (/user/v3/emails/*)  ──gRPC──► email-api
SendGrid webhook ─────────────────────────────────► email-api
Guest click/open/unsubscribe ─────────────────────► email-api
Scheduler microservice (SQS callback) ────────────► email-api
Workflow (monolith) ──gRPC──► email-api

email-api ──Kafka──► monolith consumers (activities, notifications,
                     filter-index projection, web tracking)
```

**email-api owns**, per account: emails, campaigns, envelopes, recipients, recipient events, clicks, opens, unsubscribes, links, templates, sender addresses, sender domains, the exclusion list, statistics — plus the SendGrid account relationship.

**The monolith retains:** contacts, contact groups, users, files, workflows, activities, notifications, feature subscriptions, `Feature\Usage` and all billing, and all transactional mail.

**Metering inverts:** email-api becomes the source of the dispatched-recipient metric; the monolith reads it over gRPC. See §5.5.

---

## 5. Boundary decisions

These bind every wave.

### 5.1 Who resolves recipients and renders placeholders? ⚠️ BLOCKS WAVE 0

Sending needs contact data: address, person in charge, and every field a template placeholder references. Today that is a SQL join inside the account DB.

- **Option A — monolith resolves, pushes materialised recipients.** The monolith expands contacts and groups, evaluates the exclusion list, supplies placeholder values, and sends email-api a concrete list (`contact_id`, address, resolved values). email-api never reads contacts.
- **Option B — email-api resolves via a contacts adapter**, calling the lead/contact service over gRPC.

**Recommendation: A.** Keeps contact ownership unambiguous, avoids a runtime dependency on the hottest table in the system, makes the send contract a pure value object. Cost is payload size on large campaigns — bounded by chunking the push. Option B additionally drags group expansion, dynamic group evaluation and placeholder validation into email-api.

### 5.2 How do `Contact::emails()` / `Contact::envelopes()` survive?

Cross-DB Eloquent relations cannot. In order of preference:

1. **Monolith-side read projection** fed by Kafka — a narrow table (`contact_id`, `email_id`, `envelope_id`, event type, timestamp) serving the filter index, export and the `getEmailIdsByEvent` family. Eventually consistent; fine for filtering and reporting.
2. **gRPC read path** for the few call sites needing read-after-write.

Identify which call sites genuinely need strong consistency before Wave 3.

### 5.3 Who owns SendGrid?

**email-api** — taken in Wave 2 (identity/domains) and Wave 3 (webhook + events). Whoever owns the webhook owns event ingestion. Until Wave 3 completes the webhook stays at the monolith and **forwards**, so SendGrid is repointed once, not twice.

Remember §1's SendGrid gap: this is net-new capability in email-api.

### 5.4 What happens to in-flight state at cutover?

Per account at flag-flip there may be: emails in `processing`, `ProcessEmailChunkJob` chains mid-flight, messages in SQS `scheduler-on-demand`, a live Redis counter. **Drain, don't switch** — block new sends, wait for `processing` to clear, migrate `scheduled` rows explicitly, then flip.

### 5.5 Who owns the usage metric?

**email-api computes and exposes the distinct dispatched-recipient count; the monolith keeps `Feature\Usage`, invoicing and all billing logic.**

`FeatureUsage.Show` is extended to return CRM dispatched-recipient counts alongside the existing inbox count, and `Email\UsageManager::getUsageAmount()` changes from *(local SQL + local workflow + remote inbox)* to *(remote total + local workflow notifications)*.

The alternative — monolith counts locally over a projection — is rejected: a distinct count over a projection that may lag or drop events is a **silent billing error**, strictly worse than a failed gRPC call.

**Rollout rule:** billing-affecting, so **no flag-flip cutover**. Parallel-run reconciliation for at least one full billing cycle per account, with every daily bucket matching exactly, before legacy removal. Full detail in [[Email Feature Usage Metering]].

---

## 6. Invariants — how users keep working

1. **The monolith stays the façade.** `/user/v2/emails/*` URLs, request shapes and response shapes do not change. Controller bodies become `EmailClient\Handlers\*` calls. **The frontend changes zero times.** Moving it to `/v3` is separate follow-up work.
2. **Per-account feature flags gate every cutover.** Database-per-account + the existing `FeatureFlag` mechanism = migrate and roll back one account at a time. One flag per wave.
3. **Every table group follows the same five steps:** dual-write → backfill → verify → read cutover → stop legacy write → drop. Each independently revertible; nothing dropped until the *following* wave is stable.
4. **Events replace joins.** email-api emits Kafka; the monolith maintains read projections. Consumers are rewired to events **before** the corresponding write path moves.
5. **Shadow comparison before every read cutover.** Read both sides, log mismatches, cut over at zero mismatches for a full business cycle.
6. **Billing-affecting metrics are parallel-run, never flag-flipped.** A mismatch is a release blocker, not a warning.

---

## 7. Risks

| Risk | Mitigation |
|---|---|
| Bulk send correctness at scale (missed/duplicated recipients) | Wave 5 only after 2/3/4 stable; idempotency key per `(email_id, contact_id)`; shadow-run a real campaign on a pilot account and diff envelope counts |
| SendGrid is net-new in email-api | Treat Wave 2 as feature development, not a port; spike subuser + domain-auth + webhook against a sandbox SendGrid account before committing dates |
| Eventually-consistent filter index changes user-visible results | Measure current lag first — the SQL path is already async via index rebuild; quantify the delta rather than assume regression |
| Payload size for very large campaigns under §5.1 A | Chunk the recipient push at the existing `config('mail.bulk.chunk_size')` |
| Scheduled emails lost at cutover | Explicit migration of `scheduled` rows + SQS drain in the Wave 4 runbook; inventory before, verify after |
| Two systems writing engagement data during dual-write | Single writer per table group always; dual-*write* = monolith writes both, never two services writing independently |
| Usage metering drifts → wrong invoices | email-api owns the `DISTINCT` (§5.5); parallel-run with per-day equality as the gate |
| Historical usage recomputation changes after backfill | Backfill preserves `recipient_id` + `occurred_at` identity; replay `ReconcileDailyUsageJob` on both sides and diff |
| SendGrid invalid-domain credits stop decrementing | Credits move with the dispatch path in Wave 4; owner decided before Wave 2 closes |
| Storage usage jumps when `emails` rows leave | Storage metering switches to gRPC count in the same release as the table cutover |
| Workflow email actions break | Wave 4 includes `EmailActionExecutor`; validity checks become gRPC reads in the same wave |
| Migration stalls half-done | Each wave ships value and is independently completable; no wave depends on a later one |

---

## 8. Open questions

1. **§5.1 — option A or B?** Blocks Wave 0.
2. Which `Contact::envelopes()` call sites need read-after-write consistency?
3. Do core (Digima) templates move at all? They are **not** per-account data, so they do not fit email-api's database-per-account model. Keep in the monolith core DB with email-api reading over gRPC, or give email-api a shared non-tenant schema?
4. Does the statistic DB move, or does email-api emit into the existing one?
5. Who owns the SendGrid invalid-domain credit counter after Wave 4 — email-api (which will own domain-auth state) or the monolith `Account` settings where it lives today?
6. Storage metering counts `emails` rows as storage units. Once `emails` lives in email-api, should that come from `FeatureUsage.Show`, or should storage metering be redefined in actual bytes?
7. Target account cohorts and pilot accounts per wave.
8. Does `/user/v3/emails/*` become the long-term public shape, and is an FE migration project planned to follow?
