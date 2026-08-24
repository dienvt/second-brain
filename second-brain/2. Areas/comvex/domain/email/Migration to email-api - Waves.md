---
tags: [comvex, digima, backend, domain, email, migration, microservice, runbook]
project: digima-backend-app, digima-backend-email-api
source: docs/migrations/email-domain-to-email-api/01-migration-steps.md
created: 2026-08-19
---

# Migration to email-api — Waves

The ordered, step-by-step plan. Assumes [[Migration to email-api - Overview]] — read it first for scope, boundary decisions and invariants.

## TL;DR

| Wave | Scope | Blocked by | Risk |
|---|---|---|---|
| **0** | Proto, schema, event contracts, projections, adapters, flags, shadow harness, backfill tooling | Overview §5.1 | — |
| **1** | Templates | Wave 0 | Low — the proving ground |
| **2** | Sender identity + **SendGrid as a new transport** | Wave 0 | Medium — net-new capability |
| **3** | Tracking, engagement ingestion, **metering** | Wave 0 events + projections | **High — touches invoices** |
| **4** | Classic send + workflow email action | Waves 2, 3 | High |
| **5** | Bulk orchestration | Wave 4 | **Highest — duplicate mail to customers** |
| **6** | Statistics, cleanup, public shape | Wave 5 | Low |

Waves 1–3 can run in parallel across teams. 4 → 5 → 6 are strictly sequential. No wave depends on a later one, so the migration can pause between any two waves without leaving users broken.

---

## The two reusable patterns

Defined once, referenced by every wave.

### Pattern A — moving a table group

| Step | Action | Reverts by |
|---|---|---|
| **A1. Dual-write** | Monolith writes its own tables *and* calls email-api on every mutation. email-api is not read from. | Turning off the gRPC call |
| **A2. Backfill** | Copy historical rows per account, preserving primary keys, foreign keys and timestamps **exactly**. Idempotent, resumable. | Truncating email-api tables for that account |
| **A3. Verify** | Shadow-read both sides on every read path; log mismatches with account, resource, field, both values. Run to zero mismatches for a full business cycle. | N/A — read-only |
| **A4. Read cutover** | Flip the per-account flag: reads come from email-api. Writes still dual. | Flipping the flag back |
| **A5. Stop legacy write, then drop** | (a) monolith stops writing its tables. (b) After the *following* wave is stable, drop them. | Between a and b: re-enable legacy write, re-backfill from email-api |

> **Never skip A3. Never combine A4 and A5 in one release.**

### Pattern B — per-account cutover runbook

Applies at A4 in every wave. Database-per-account is what makes this safe.

1. **Pre-check** — no emails in `processing`; scheduled emails inventoried; no chunk jobs in flight; no Redis progress counters; shadow mismatch count zero for the trailing cycle.
2. **Quiesce** (send-path waves) — block new sends for the account. Seconds to minutes, not a maintenance window; invisible to anyone not mid-send.
3. **Drain** — wait for in-flight work. Never switch mid-campaign.
4. **Flip** — enable the wave's flag for the account.
5. **Smoke** — exercise read paths + one real low-volume send.
6. **Watch** — error rate, gRPC status codes, queue depth, SendGrid accepted/deferred, and the wave's own metric, for one business cycle.
7. **Roll back if needed** — flip the flag off. Writes are still dual at A4, so no data is lost.

**Cohorts:** internal/QA account → one small pilot customer → 5% → 25% → 100%. Hold one full business cycle at each stage.

---

## Wave 0 — Foundations

**Goal:** everything later waves need, zero user-visible change.

> ⚠️ **Blocked by overview §5.1** (who resolves recipients) — it determines the shape of every proto. §5.5 (usage metric ownership) must be settled before Wave 3 design begins.

1. **Extend `digima-backend-email-proto`** — new services `Templates`, `CoreTemplates`, `Addresses`, `Domains`, `Exclusions`, `Recipients`, `RecipientEvents`, `Unsubscribes`, `Campaigns`, `Statistics`; extend `FeatureUsage` for CRM dispatched-recipient counts. Follow Comvex proto conventions (`List` RPC pagination, optional filters, enum naming). Public API from day one.
2. **Per-account schema in email-api** for the ~18 tables, provisioned through the existing `Databases` service so new accounts get them automatically. Preserve column names, types and nullability — the backfill is a straight copy and any coercion becomes a data bug.
3. **Kafka event contracts** for everything monolith consumers currently derive from SQL: email created/updated/deleted, campaign started/progressed/completed, envelope created, envelope dispatched, delivery/bounce/failure, open, click, unsubscribe. Extend the existing `email/event/*` proto tree — don't invent a parallel scheme.
4. **Monolith read projections** replacing `Contact::emails()` / `Contact::envelopes()` (overview §5.2), plus their Kafka consumers. Build now, keep dark; they become the read source in Wave 3.
5. **Adapter layer** — new `EmailClient\Handlers\*` mirroring today's model API surface so controllers and jobs swap one line at a time instead of being rewritten. Reuse `BaseHandler` unchanged.
6. **Feature flags** — `EMAIL_API_TEMPLATES`, `EMAIL_API_SENDER`, `EMAIL_API_TRACKING`, `EMAIL_API_SEND`, `EMAIL_API_BULK`, `EMAIL_API_STATISTICS`. On `FeatureFlag`, resolvable per account.
7. **Shadow-compare harness** — reusable helper that reads both sides, diffs, logs structured mismatches. Every A3 uses it; without it A3 is guesswork.
8. **Backfill tooling** — one idempotent, resumable, per-account command with progress reporting and dry-run. Every A2 uses it.

**Done when:** proto published and consumed; email-api provisions the full schema for a new account; harness and backfill exercised end-to-end on QA; flags exist, default off. No production behaviour changed.

---

## Wave 1 — Templates

**Why first:** a leaf domain. No send path, no engagement data, no billing. Where the Pattern A/B machinery gets proven against real user traffic at low risk.

**Tables:** `email_templates`, `email_template_folders`, `email_template_labels`, `email_template_label_pivot` (account DB) + core-DB equivalents.

1. **Resolve overview open question 3 first** — do core (Digima) templates move at all? They aren't per-account and don't fit email-api's database-per-account model. Either keep them in the monolith core DB and have email-api read over gRPC, or give email-api a shared non-tenant schema. **Decide before writing the `CoreTemplates` proto.**
2. Pattern A1–A5 for the account tables.
3. Swap controller bodies behind the flag, request/response shapes identical: `User/v2/Email/Template/Account/*`, `User/v2/Email/Template/Core/*`, `Admin/v2/Email/Template/*`, `Internal/v3/CoreEmailTemplate*`.
4. Move the template jobs (`Jobs/Email/Template/**`, ~20 files) to gRPC calls.
5. Rewire workflow coupling: `Workflow/Action/CreateActionRequest` validates a template reference; `CloneWorkflowJob` and `UpdateWorkflowJob` read template rows → gRPC reads.
6. Keep `Policies/Email/Template/*` in the monolith — authorisation stays at the façade.
7. Delete monolith models, jobs, events, transformers at A5b.

**Done when:** all template reads/writes for 100% of accounts hit email-api; monolith template tables dropped; workflow template validation works through gRPC.

---

## Wave 2 — Sender identity (+ SendGrid)

**Why here:** the send path depends on it, and it's where SendGrid ownership transfers.

> ⚠️ **This is feature development, not a port.** email-api has no SendGrid support at all — its `Provider` is SMTP/IMAP only. Spike subuser creation + domain authentication + webhook receipt against a sandbox SendGrid account before committing dates.

1. Pattern A1–A5 for `email_addresses` and `email_domains`.
2. Build SendGrid support in email-api, porting the monolith's managers: `Sendgrid\AccountManager`, `Sendgrid\DomainManager`, `Sendgrid\SubUserManager`, `Sendgrid\UpdateSuppressionPermissionManager`. Credentials into email-api's Vault; monolith SendGrid credentials removed at A5b, **not before**.
3. Move `Jobs/Email/Domain/ValidateDomainJob` (DNS verification) and `Console/Commands/Email/{CheckSendgridApiKeyStatusCommand,UpdateSendgridKeyCommand}`.
4. **Decide invalid-domain credit ownership** (overview open question 5). Lives in `Account::SENDGRID_SETTING_SENT_FROM_INVALID_DOMAIN_CREDITS` today, decremented per dispatch. Moving it to email-api is cleaner — the "dirty IP" reassignment it triggers is a SendGrid subuser operation that will already live there. `Jobs/Account/ResetSendGridDomainCreditsJob` and `Schedulers/Email/ResetInvalidDomainCreditsScheduler` follow the counter.
5. Swap `User/v2/Email/{Addresses,Domains}Controller` and `Email/v2/AddressesController` behind the flag.
6. Keep `Support/Validators/EmailRecipientValidator` and `Models/Core/FreeEmailDomain` in the monolith — used by non-email validation paths.

**Done when:** addresses and domains managed entirely in email-api; email-api holds the SendGrid subuser relationship and API keys; domain auth succeeds end-to-end for a new account; the credit counter has one unambiguous owner.

---

## Wave 3 — Tracking, engagement ingestion, and metering

**The pivotal wave.** email-api becomes the writer of engagement data and the monolith becomes a reader — which is also where the billing metric moves.

> ⚠️ `email_envelope_recipient_events` **is** the billing source. Metering is not follow-up work; it is part of this wave. See [[Email Feature Usage Metering]].

**Blocked by:** Wave 0 steps 3–4 complete and consuming; overview §5.5 decided.

1. Pattern A1–A5 for the engagement tables. `email_envelope_recipient_events` is the highest-volume table in the domain — **size the backfill per account before starting**; expect it to be the longest-running task in the migration.
2. **Move the exclusion list** (suppression). Sending consults it, so it must be readable from email-api before Wave 4.
3. **Webhook** — keep `POST /sendgrid/v2/events` at the monolith and **forward** to email-api rather than repointing SendGrid twice. Move `ParseRawSendgridEventsJob` → `StoreSendgridEventsForAccountJob` logic into email-api. Repoint SendGrid directly only at A5, after forwarding has been stable.
4. **Guest endpoints** — `GET /email/v2|v3/click`, `/open`, `POST /email/v2/unsubscribe`. These live in emails already delivered to real inboxes and must keep working **indefinitely**, not just through the migration. Keep the monolith URLs as permanent forwarders. Preserve the rate limits (`throttle.any:20,60`, `throttle.email_opens:1,3600`).
5. **Rewire monolith consumers from SQL to events — before A4:**
   - `Listeners/Email/Envelope/UpdateStatisticSubscriber`
   - `Listeners/Email/Envelope/NotifyBouncedRecipientsSubscriber`, `UnBounceEmailAddressListener`
   - `Listeners/Email/{Bulk,Classic,Inbox}/LogActivitySubscriber`, `Services/Activities/MetaEnricher`
   - `Notifications/Email/*` (opened, clicked, bounced, digests)
   - `FilterClient\Handlers\ContactAllFieldsIndex` `email_events` → projection
   - `Services/Web/Tracking/TrackingManager` envelope correlation
   - `Contact::{emails,envelopes,getEmailIdsByEvent,getEnvelopeEventQuery,getEnvelopeRecipientEventQuery}` → projection
   - `Export/Contact/{CreateExportRequestHelper,RowParser}`
   - `EventProducer\Handlers\{EmailEnvelopeClicks,RecipientEvents}` — either retire in favour of email-api emitting directly, or keep the monolith as relay. **Decide explicitly**: duplicate producers on one topic corrupt downstream consumers.
6. **Metering — parallel run, not a flag flip.** Full procedure in [[Email Feature Usage Metering]]. In short: extend `FeatureUsage.Show` with the CRM distinct dispatched-recipient count; shadow-compute alongside the legacy local count; replay `ReconcileDailyUsageJob` over history and diff; switch only after a **full billing cycle of exact daily and monthly equality**, removing the legacy count in the **same** release.

**Done when:** email-api writes all engagement data; SendGrid posts directly to email-api; guest tracking works from both old and new URLs; every consumer reads events or projections; email + storage usage reconciled exactly for a full billing cycle at 100% of accounts.

---

## Wave 4 — Classic send

**Scope:** single-recipient sending — `Email` lifecycle, recipient resolution, body/link parsing, placeholder rendering, envelope creation, SendGrid dispatch — plus the workflow email action.

**Blocked by:** Waves 2 and 3. Sending needs sender identity, the exclusion list and link creation already in email-api.

1. Pattern A1–A5 for `emails`, `email_contact_pivot`, `email_contact_group_pivot`, `email_envelopes`, `email_files`.
2. Implement the send path per §5.1. Under option A the monolith resolves recipients and pushes materialised recipients; email-api owns envelope creation and dispatch.
3. Move `BodyParser` (link extraction, oversized-link validation, `createEmailLinks`), `PlaceholderHelper`, `EmailLocalPartFormatter`, `EnvelopeDecorator`, `Sendgrid\{EmailSender,ResponseStatusChecker,UnBounceManager}`.
4. **Keep recipient validation at the façade.** `RecipientValidationHelper`, `Email::isValid()` and `CannotSendInvalidEmailException` reasons must surface as today's exact API error shapes — the frontend renders those reason codes.
5. **Workflow email action** — `EmailActionExecutor` calls `EnvelopeCreator` in-process today; becomes a gRPC send. It also writes `processed_at` on the action participant pivot and the email: keep the pivot write in the monolith, derive the email's `processed_at` from the send response. `MarkWorkflowInvalidSubscriber` and `Jobs/Workflow/Action/DeleteActionJob` become gRPC reads.
6. **Invalid-domain credits** wired per the Wave 2 decision, at the dispatch point, keeping the production-only guard.
7. Move `Jobs/Email/{CreateEmailJob,UpdateEmailJob,DeleteEmailJob,SendTestEmailJob}` and `Jobs/Email/Attachment/*`; swap `User/v2/Email/EmailsController` behind the flag.
8. Rewire `Services/Bulk/Email/BulkActionHelper` (bulk delete) to gRPC.
9. `Jobs/Feature/EndSubscriptionJob` and the account cancel/disable checks in `ProcessEmailJob` become gRPC calls.

**Cutover notes** — first wave where Pattern B's quiesce and drain matter. Per account: block new sends, wait for `processing` to clear, **migrate `scheduled` emails explicitly**, drain that account's SQS `scheduler-on-demand` messages, then flip. Scheduled emails are the easiest thing to lose — inventory before, verify after.

**Done when:** all classic sends for 100% of accounts run through email-api; workflow email actions send through email-api; scheduled classic sends fire correctly across the cutover; test sends, validation errors and attachments unchanged from the frontend's perspective.

---

## Wave 5 — Bulk orchestration

**Highest risk in the migration.** A correctness bug means customers send duplicate mail to their contacts, or miss recipients, at scale.

**Blocked by:** Wave 4 complete and stable.

1. **Replace inferred completion with explicit state.** Today it's a Redis counter + a `Cache::lock` race in `ProcessEmailChunkJob::dispatchSubsequentChunkJob()`, a 1-hour TTL, and a `failed()` handler that continues the chain with a zero count. Model a **campaign** in email-api with real state and per-chunk accounting instead of reproducing the counter.
2. **Idempotency per `(email_id, contact_id)`.** The single most important safeguard — it makes retries, redeliveries and partial-failure recovery safe, and is the difference between a recoverable incident and duplicate mail to a customer's contact list.
3. Port the chunking controls: `config('mail.bulk.chunk_size')`, `Setting::KEY_BULK_EMAIL` (concurrency and interval with the existing 0–500 / 0–100 clamps), `ConcurrentJobDispatcher` semantics, and the inter-chunk delay that paces SendGrid.
4. **Recipient push** under §5.1 A: chunk the materialised list at the existing chunk size so payloads stay bounded on large campaigns.
5. Port the empty-recipient case: zero contacts must still mark the email processed, not fail (see the PR #3774 note in `PrepareEmailChunkJob`).
6. Port dynamic contact groups: `AttachIncludedContactGroupsJob`, `AttachExcludedContactGroupsJob`, `PrepareBulkEmailContactGroupsSubscriber`. Group expansion stays in the monolith under option A.
7. **Scheduled bulk sending** — `scheduleEmailSending()` pushes to SQS `scheduler-on-demand` with an HTTP callback whose `{email_client_code}` placeholder the scheduler substitutes. Repoint the callback at email-api, keep the monolith endpoint as forwarder until every already-queued message has fired. `Jobs/Email/CancelScheduledSendingJob` moves with it.
8. Port `BulkEmailSentActivityAccumulator` + `FlushStaleBulkEmailSentActivityBufferJob`, or replace the buffer with campaign-progress events consumed by the monolith's activity logger.

**Cutover notes** — never flip mid-campaign; pre-check must show no active campaign. Before the first real cohort, **shadow-run a genuine campaign** on a pilot account and diff envelope counts, recipient sets and SendGrid accepted counts against the legacy path.

**Done when:** all bulk sends for 100% of accounts run through email-api; envelope and recipient counts match the legacy path exactly on shadow runs; scheduled and cancelled campaigns behave correctly across the cutover; progress and completion observable per campaign.

---

## Wave 6 — Statistics, cleanup, public shape

1. Resolve overview open question 4 — does the statistic DB move, or does email-api emit into the existing one? `envelope_statistics`, `Services/Statistic/Email/{Fetcher,Envelope/Generator}` and `Console/Commands/Statistic/UpdateEnvelopeStatisticsCommand` follow that decision.
2. Pattern A1–A5 for statistics; swap `User/v2/Email/StatisticsController`.
3. **Delete the monolith email domain** — models, jobs, services, events, listeners, transformers, migration traits, account-DB tables. Only after the owning wave has been stable in production, and only at A5b.
4. Retire what's now duplicated: `Services/Email/EnvelopeCreator`, `RecipientResolver`, `EmailContactSynchronizer`, the `*AttributeExtractor` classes, monolith SendGrid credentials.
5. Keep `/user/v2/emails/*` as a thin permanent adapter, **or** plan the FE move to `/user/v3/emails/*` as a separate project (open question 8). Guest tracking URLs stay forever regardless.
6. Write up the resulting decisions in `docs/adr/` and archive the migration docs as historical record.

**Done when:** the monolith contains no email domain code or tables beyond the façade controllers and projections; all email functionality served by email-api; transactional mail untouched and still working.

---

## Cross-wave checklist

Apply before any wave is called done.

- [ ] Proto changes reviewed as public API; backward compatible
- [ ] Feature flag exists, defaults off, resolvable per account
- [ ] Backfill idempotent, resumable, dry-runnable, sized against the largest account
- [ ] Shadow-compare ran to zero mismatches for a full business cycle
- [ ] Rollback tested — flag off, legacy path still serves
- [ ] Every consumer in overview §3.4 rewired and verified
- [ ] Activities, notifications and filter results verified against real account data
- [ ] Usage metering unaffected, or reconciled per §5.5
- [ ] Cohort rollout completed with a hold at each stage
- [ ] Legacy write disabled and tables dropped in a *later* release than the read cutover
- [ ] Tests: feature tests in email-api per the `grpc-implementation` pattern; monolith contract tests against the adapter layer; send path exercised end-to-end

## What must never break

1. **Guest tracking URLs in already-delivered mail** — `/email/v2|v3/click`, `/open`, `/email/v2/unsubscribe`. These live in customers' inboxes forever.
2. **Scheduled emails across a cutover.**
3. **Usage metering totals** — they become invoices.
4. **Transactional mail** (password reset, activation, notifications) — out of scope, must stay untouched.
5. **`/user/v2/emails/*` request and response shapes**, for as long as the frontend uses them.
