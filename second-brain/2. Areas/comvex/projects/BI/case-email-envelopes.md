# Case study — `email_envelope_recipients` vs `inbox_envelopes`

Back to [[README]]. See also [[oltp-mapping]] for where each table comes from.

## The question

> The `inbox_envelopes` table contains the email send/receive history per customer.
> However, even when there is send data to a customer in `email_envelope_recipients`,
> there is often no corresponding data in `inbox_envelopes`. Is this a data omission?

## Answer

**No — it is by design, not a data omission.**

The two tables are populated by **two independent subsystems** with different triggers,
different lifecycles, and no shared primary key. A row in `email_envelope_recipients` only
means "the user addressed an email to this contact." It does **not** imply the email was
actually sent, processed, or delivered.

## The two write paths

| Axis | `email_envelope_recipients` | `inbox_envelopes` |
|---|---|---|
| Source DB | `dgm_account` | `dgm_account_email` |
| Owner | Legacy PHP monolith `digima-backend-app` | Go microservice `digima-backend-email-api` |
| Insert site | `app/Jobs/Email/Envelope/CreateEnvelopeJob.php:293` (`$envelope->addRecipient()`) | `interface/listeners/envelope/create.go:36` → `domain/services/email/envelope.go:CreateFromEmail` (~line 199) |
| Trigger | Laravel job runs when a user **composes** an email. Creates one row per TO/CC/BCC. | Event listener fires when an `Email` transitions to `StatusProcessed` (outbound send complete) OR when an inbound message lands (IMAP poll / SendGrid inbound webhook / Gmail sync). |
| Semantics | **Intent to send** | **Actual email event** |
| BI extract filter | none | `envelope_addresses.resource_type = ENVELOPE_ADDRESS_RESOURCE_TYPE_CONTACT` AND `ExcludeDeleted: true` |

## Why the two do not share a key

The legacy monolith does call the new service — but the bridge carries **no legacy IDs**.

```
[legacy] EmailsController.store (digima-backend-app/.../EmailsController.php:57)
          writes to dgm_account.emails / email_envelopes / email_envelope_recipients
          → gRPC EmailsService.Create (digima-backend-email-proto/email-api/email/api/v1/emails.proto:13)
            request carries: subject, content, sender (user_id, address_id),
                             recipients (user IDs), scheduled_at, options
            request does NOT carry: legacy email_id, envelope_id, recipient_id, code

[new]    email-api receives the call
          → INSERT dgm_account_email.emails
          → on StatusProcessed, CreateFromEmail listener fires
          → INSERT dgm_account_email.envelopes
              id         — fresh PK
              code       — fresh utils.GenerateUID(16)
              message_id — freshly generated RFC 5322 Message-ID
          → INSERT dgm_account_email.envelope_addresses (one per addressee)
```

So the identifiers on the two sides are **independently minted**. There is no outbox, no
Kafka consumer on the legacy side, no cron sync, no DB trigger. Envelope created events
ARE published to Kafka (`digima-backend-email-api/infra/producer/kafka/client.go:81`) but
nothing in `digima-backend-app` subscribes.

A FIXME at `digima-backend-email-api/infra/worker/client.go:78` hints the long-term direction
is to retire the legacy path, not to bridge it.

## Seven by-design reasons a recipient has no envelope

All of these are expected — none need fixing:

1. **Draft / scheduled never sent** — recipient was inserted at compose time; the email
   never reached `StatusProcessed`.
2. **Pre-microservice history** — for emails created before `digima-backend-email-api`
   existed, `dgm_account_email.envelopes` was never populated. Pin the cutoff with
   `SELECT MIN(created_at) FROM dgm_account_email.envelopes;`
3. **Validation failure** — invalid address, suppression/exclusion list, placeholder error.
   Legacy records the recipient and marks the send failed; the new service never sees it.
4. **Send aborted / held** — rate limit, fraud check, daily cap. Queued but never
   processed.
5. **Integration / external flow** that bypasses the gRPC send path entirely.
6. **Recipient is an internal `USER`**, not a `CONTACT`. The BI def for `inbox_envelopes`
   filters to `resource_type = CONTACT`, so envelopes whose addressees are users get dropped.
7. **Soft-deleted envelope**. `ExcludeDeleted: true` on the BI def drops them.

## What a real bug would look like

Rule these out before labeling any specific gap "by design":

- **BI loader miss:** envelope exists in `dgm_account_email.envelopes` with `deleted_at IS
  NULL` and an `envelope_addresses` row where `resource_type = CONTACT` matches — but no
  row in Redshift `inbox_envelopes`. Action: rerun `Jenkinsfile-Digest` with `from_date`/`to_date`
  covering the gap.
- **App-side bug:** `Email.status = Processed` in `dgm_account_email` after the rollout
  date, but no `envelopes` row written. The listener `CreateFromEmail`
  (`domain/services/email/envelope.go:199`) didn't fire or errored silently. Check
  email-api logs.
- **Windowing artifact:** gaps exist only on yesterday's data. Stage-1 is nightly
  incremental; yesterday is usually half-loaded. Not a bug. Query against data ≥2 days old.

## Decision tree

```
Q1. In Redshift, recipients row with NO matching inbox_envelopes?
    No  → not reproducible, stop.
    Yes → Q2.

Q2. In dgm_account_email, does envelopes ⋈ envelope_addresses
    (resource_type=CONTACT, resource_id=contact_id) have a row in window?
    Yes + deleted_at IS NULL     → BI LOADER BUG. Rerun Jenkinsfile-Digest.
    Yes + deleted_at IS NOT NULL → BY DESIGN (#7, ExcludeDeleted).
    No                           → Q3.

Q3. In dgm_account, what is the parent email's status?
    (Join: email_envelope_recipients.envelope_id
           → email_envelopes.id → email_envelopes.email_id → emails.id)
    draft / scheduled                         → BY DESIGN (#1).
    failed / cancelled / aborted              → BY DESIGN (#3, #4).
    sent, email.created_at < rollout date     → BY DESIGN (#2).
    sent, email.created_at ≥ rollout date     → APP-SIDE BUG. Check listener logs.
```

## Verification SQL

### Redshift (start here — one query often settles it)

```sql
-- Count unmatched recipients per legacy email status per week.
-- If the histogram is dominated by draft/scheduled/failed, it's by design.
SELECT COALESCE(e.status, '<no_legacy_email_row>') AS legacy_status,
       DATE_TRUNC('week', r.created_at)            AS week,
       COUNT(*)                                    AS unmatched_count
FROM   email_envelope_recipients r
LEFT JOIN email_envelopes ee ON ee.id = r.envelope_id AND ee.account_id = r.account_id
LEFT JOIN emails          e  ON e.id  = ee.email_id   AND e.account_id  = r.account_id
LEFT JOIN inbox_envelopes ie
       ON ie.account_id = r.account_id
      AND ie.contact_id = r.contact_id
      AND ie.occurred_at BETWEEN r.created_at - INTERVAL '2 hours'
                             AND r.created_at + INTERVAL '2 hours'
WHERE  r.account_id = :account_id
  AND  r.created_at::date BETWEEN :from_date AND :to_date
  AND  ie.id IS NULL
GROUP BY 1, 2
ORDER BY week DESC, unmatched_count DESC;
```

### OLTP `dgm_account_email` (new email microservice)

```sql
-- Does the envelope exist server-side?
SELECT e.id, e.message_id, e.type, e.code, e.occurred_at,
       e.contact_id, e.deleted_at, e.created_at, e.updated_at,
       ea.type AS address_type, ea.resource_type, ea.resource_id, ea.envelope_email
FROM   envelopes e
JOIN   envelope_addresses ea ON ea.envelope_id = e.id
WHERE  ea.resource_type = 'ENVELOPE_ADDRESS_RESOURCE_TYPE_CONTACT'
  AND  ea.resource_id   = :contact_id
  AND  e.occurred_at BETWEEN :recipient_created_at - INTERVAL 2 HOUR
                         AND :recipient_created_at + INTERVAL 2 HOUR
ORDER BY e.occurred_at;

-- Rollout cutoff — any legacy email before this date is pre-microservice.
SELECT MIN(created_at) AS first_envelope FROM envelopes;
```

### OLTP `dgm_account` (legacy monolith)

```sql
SELECT r.id AS recipient_id, r.envelope_id, r.contact_id, r.type, r.email_address,
       r.created_at,
       ee.id AS envelope_row_id, ee.email_id,
       e.status AS legacy_email_status, e.processed_at, e.scheduled_at,
       e.created_at AS email_created_at, e.type, e.category
FROM   email_envelope_recipients r
JOIN   email_envelopes ee ON ee.id = r.envelope_id AND ee.account_id = r.account_id
JOIN   emails           e ON e.id  = ee.email_id   AND e.account_id  = r.account_id
WHERE  r.id         = :recipient_id
  AND  r.account_id = :account_id;
```

## What to tell analytics

- `email_envelope_recipients` = **intent log** (legacy DB, written at compose time).
- `inbox_envelopes` = **actual-event log** (new service DB, written on send/receive).
- There is no cross-system primary key — the gRPC bridge doesn't carry legacy IDs.
- The canonical "was this email actually delivered?" signal is `emails.status` from the
  new email service, **not** the presence of an `inbox_envelopes` row.
- If they need an audit-grade recipients ↔ envelopes join, that's a product change
  (propagate an ID through the gRPC call, or retire the legacy path). Worth a YouTrack
  ticket if the need is recurring.

## Key files (for grepping later)

| Concern | Path |
|---|---|
| BI def: `email_envelope_recipients` | `digima-mgmt/jenkins/scripts/BI-New/definitions/daily/email_envelope_recipients.json` |
| BI def: `inbox_envelopes` | `digima-mgmt/jenkins/scripts/BI-New/definitions/daily/inbox_envelopes.json` |
| Legacy write (PHP) | `digima-backend-app/app/Jobs/Email/Envelope/CreateEnvelopeJob.php:293` |
| Legacy send entry point | `digima-backend-app/app/Http/Controllers/User/v3/Email/EmailsController.php:57` |
| Legacy gRPC handler → new service | `digima-backend-app/app/Services/EmailClient/Handlers/Emails.php:95` |
| gRPC proto | `digima-backend-email-proto/email-api/email/api/v1/emails.proto:13` |
| New service envelope create | `digima-backend-email-api/domain/services/email/envelope.go` (`CreateFromEmail`, ~line 199) |
| New service listener | `digima-backend-email-api/interface/listeners/envelope/create.go:36` |
| Unconsumed Kafka producer | `digima-backend-email-api/infra/producer/kafka/client.go:81` |
| Envelope schema migrations | `digima-backend-email-api/infra/database/migrations/mysql/003_create_envelopes_table.up.sql`, `014_create_envelope_addresses_table.up.sql` |
| `resource_type` / address-`type` enum constants | `digima-backend-email-api/domain/models/email/envelope_address.go:7-13` |

## Confluence references (the canonical naming)

The docs call the two systems "Email (Old Classic & Bulk)" and "Email Inbox". Useful
references:

| Page | ID | Why |
|---|---|---|
| [Billing Microservice - AWAR](https://comvex.atlassian.net/wiki/spaces/SE/pages/2585624588) | 2585624588 | Defines the three billable sources that make up `email_message`: "Email (Old Classic & Bulk) Envelope Count", "Workflow Notification Action Pivot Count (Processed)", "Email Inbox (Sent Only & Not Imported from Sync) Envelope Count". Also the Kafka topic catalog: `email.recipient_events.created.v1` (old, Backend App) vs `email.envelopes.created.v1` (new, Email API). The `email.envelopes.deleted.v1` payload has an `is_legacy (bool)` field — proof the systems are deliberately kept distinguishable. |
| [Email API](https://comvex.atlassian.net/wiki/spaces/SE/pages/1940160513) | 1940160513 | New service overview. Go 1.19, gRPC, MySQL 8. (Stub doc — notes it will be deleted in favor of repo README.) |
| [Email Inbox OAuth2](https://comvex.atlassian.net/wiki/spaces/SE/pages/2092040195) | 2092040195 | Why the new service had to exist: Microsoft/Google deprecated IMAP/SMTP basic auth. New routing keys `v2.envelopes.sync_oauth2_by_id` / `_by_date_range` / `_by_next_ids`, new `Authorization` resource, new `CONNECTION_AUTHORIZATION_TYPE_OAUTH2_MICROSOFT`. |
| [Email Inbox GraphQL query structure](https://comvex.atlassian.net/wiki/spaces/SE/pages/2475032629) | 2475032629 | New data model — `InboxEnvelopeType = SENT | RECEIVED`, `InboxAddressResourceType = USER | CONTACT` (matches the source-code enum exactly), `InboxEmailStatus = DRAFT | SENT | SCHEDULED`. |
| [Postmortem - Inbox Integration with Userweb BFF](https://comvex.atlassian.net/wiki/spaces/SE/pages/2559148078) | 2559148078 | 14-week project released 2025-04-16. Explicit goal: "Eliminate usage of Email Inbox resource in the User API Backend App." Inbox read path moved to Userweb BFF → email-api. Compose/send on legacy still delegates via gRPC. |

### Why the old `email_message` billing source is a UNION of three

From the Billing AWAR table:

| Billing source | Owning system | Applies to |
|---|---|---|
| Email (Old Classic & Bulk) Envelope Count | Old monolith | Legacy composed outbound + bulk sends |
| Workflow Notification Action Pivot Count (Processed) | Old monolith (Laravel workflow engine) | Workflow-triggered email notifications |
| Email Inbox (Sent Only & Not Imported from Sync) Envelope Count | New email-api | Only the `type=SENT` envelopes where `is_imported=false` — the imported 3-year historical sync is excluded from billing |

This is exactly why `inbox_envelopes` feels "missing" relative to `email_envelope_recipients`:
- The new service's billing view only counts a subset of its own rows.
- Workflow-driven emails never create rows in `inbox_envelopes` at all — they stay on the legacy side only.
- Received inbox envelopes (the inbox's actual purpose) have no corresponding recipient in the legacy `email_envelope_recipients` either.
