---
tags: [comvex, digima, backend, domain, email, billing, metering, usage]
project: digima-backend-app, digima-backend-email-api
source: app/Services/Email/UsageManager.php, app/Services/Storage/UsageManager.php, app/Services/EmailClient/Handlers/FeatureUsage.php, app/Console/Commands/Email/CalculateStorageOverchargeCommand.php
created: 2026-08-19
---

# Email Feature Usage Metering

How email usage is metered and billed, and why it is the most constrained part of [[Migration to email-api - Overview|moving the email domain to email-api]].

Related: [[Feature Usage]] (the `feature_usage` table and daily-bucket mechanics) · [[Usage mismatch]] · [[Email usage]] · [[Invoice flow]] · [[Email Tracking - Delivery, Open, Click]]

## TL;DR

**Email usage metering is already a hybrid across two systems**, and it feeds invoicing.

| # | Source | Where it lives | Shape |
|---|---|---|---|
| 1 | `email_envelope_recipient_events` where `name = dispatched` | monolith account DB | **`COUNT(DISTINCT recipient_id)`** |
| 2 | `workflow_action_participant_pivot` × `workflow_actions` where `type = notification`, on `processed_at` | monolith account DB | row count |
| 3 | `FeatureUsage.Show(from, to)` → `getEnvelopes()->getResourceCount()` | **email-api, over gRPC** | count (inbox sends) |

Sum → `Feature\Usage` under `Account::BILLING_PROPERTY_EMAIL`, as a monthly `amount` **and** daily buckets via `addDailyUsageAt`.

> **There is no send-time quota enforcement.** Metering is entirely post-hoc. The only send-time gates are the `email` feature subscription and the account status checks in `ProcessEmailJob`.

---

## 1. The email usage metric

`app/Services/Email/UsageManager.php` extends `AbstractUsageManager`:

```
updateUsage($at)
 ├─ find or make Feature\Usage for BILLING_PROPERTY_EMAIL at $at
 ├─ [$startOfMonth, $endOfMonth] = get_month_bounds($usageDate, 'UTC')
 ├─ [$startOfDay,   $endOfDay]   = get_day_bounds($usageDate, 'UTC')
 ├─ $usage->amount = getUsageAmount($startOfMonth, $endOfMonth)
 ├─ $usage->addDailyUsageAt($usageDate, getUsageAmount($startOfDay, $endOfDay))
 └─ save

getUsageAmount($from, $to)
 ├─ (1) Event::where('name', dispatched)->whereBetween('occurred_at', …)
 │        ->select('recipient_id')->distinct()->count('recipient_id')
 ├─ (2) workflow_action_participant_pivot join workflow_actions
 │        where type = notification and processed_at in range  → count
 ├─ early return (1)+(2) if no active `email` subscription after $from
 └─ (3) FeatureUsage.Show($from, $to)->getData()->getEnvelopes()->getResourceCount()
        (swallows EmailClientForbidden / FeatureForbidden / NotImplemented → 0)
```

**Workflow notifications count as email usage** — that's source 2. It's easy to miss, and it stays in the monolith after the migration because workflows stay in the monolith.

### Driven by

- `Jobs/Feature/CalculateFeatureUsageJob` → `Jobs/Feature/ReconcileDailyUsageJob`
- `Jobs/Feature/CalculateUsageJob`
- `Migration/Traits/FeatureUsageSetup` during account provisioning

### Consumed by

`Billing\SegmentMaker\UsageBased\EmailSegmentMaker` → `Billing\InvoiceCalculator`, `Billing\ReconcileUsageService`

### On the time windows

The code passes `'UTC'` to `get_month_bounds` / `get_day_bounds`, while the stored `daily_usage` keys in `feature_usage` are `15:00:00 → 14:59:59` windows — i.e. JST day boundaries (see the worked example in [[Feature Usage]]). Treat the exact semantics as something to **verify empirically, not reason about**: any migration must reproduce the same bucket keys byte-for-byte.

---

## 2. Storage metering also counts email data

`app/Services/Storage/UsageManager.php`, under `Account::BILLING_PROPERTY_STORAGE`:

- `buildEmailResourceQuery($to)` → `Email::query()->where('created_at', '<=', $to)` — **counts `emails` rows as storage units**
- plus file usage and file-management usage
- plus inbox usage via the same `FeatureUsage.Show` gRPC call

### Ghost storage overcharge

`Console/Commands/Email/CalculateStorageOverchargeCommand` calls `FeatureUsage.GetGhostStorage($monthEnds)` to get cumulative orphaned-attachment bytes per month, then corrects recorded storage usage:

```
correctBytes = max(0, totalStorage - ghostBytes)
```

Per month, in the billing timezone, against `Usage::findByFeatureAt(BILLING_PROPERTY_STORAGE, $dateInBillingTz)`.

---

## 3. SendGrid invalid-domain credits

Not billing, but metered at exactly the same point in the send path.

`Sendgrid\AccountManager::decrementSentFromInvalidDomainCredits()` is called from `DispatchEnvelopeJob` **in production only**, for every envelope sent from an unauthenticated sender domain:

```
credits = Account::SENDGRID_SETTING_SENT_FROM_INVALID_DOMAIN_CREDITS
credits--                       → save on the account
if credits == 0 → updateSubUserIps('dirty')   // reassign subuser to dirty IP pool
```

Reset by `Jobs/Account/ResetSendGridDomainCreditsJob` / `Schedulers/Email/ResetInvalidDomainCreditsScheduler`.

This is a **sender-reputation control**: keep sending from unauthenticated domains and your account's SendGrid subuser is moved to shared "dirty" IPs.

---

## 4. Why this constrains the migration

The tracking wave moves `email_envelope_recipient_events` — **the billing source**. So metering cannot be follow-up work; it is a gate on that wave.

### Four hard constraints

1. **Source 1 is a distinct count, not a sum.** It cannot be reconstructed by adding per-chunk counts, and it cannot be computed by adding two independent sources without risking double-counting the same recipient. **Whichever service holds the table must do the `DISTINCT` itself.**
2. **Historical windows must stay answerable.** `updateUsage($at)` and `ReconcileDailyUsageJob` recompute arbitrary past periods. The backfill must preserve `recipient_id` and `occurred_at` identity exactly — otherwise **historical invoices change**.
3. **Daily *and* monthly granularity both matter**, and the bucket-key semantics (§1) must be preserved exactly.
4. **Storage metering breaks too.** Once `emails` rows leave the monolith, `Storage\UsageManager`'s row count has nothing to count.

### The decision

**email-api computes and exposes the distinct dispatched-recipient count; the monolith keeps `Feature\Usage`, invoicing and all billing logic.**

`FeatureUsage.Show` gains CRM dispatched-recipient counts alongside the existing inbox count, and `getUsageAmount()` changes:

| | Sources |
|---|---|
| **Before** | local SQL distinct + local workflow notifications + remote inbox |
| **After** | **remote total** + local workflow notifications |

The alternative — the monolith keeps a Kafka-fed projection and counts locally — is **rejected**: a distinct count over a projection that may lag or drop events is a *silent billing error*, strictly worse than a failed gRPC call you can see and retry.

### The rollout rule: parallel run, never a flag flip

Billing-affecting metrics do not get the per-account flag-flip treatment the rest of the migration uses.

1. Extend `FeatureUsage.Show` to answer for **arbitrary historical windows**.
2. Add a shadow computation in `getUsageAmount()`: legacy local count stays the **recorded** value; compute the remote count alongside; log the delta per account per day.
3. Replay `ReconcileDailyUsageJob` over historical periods against both sides and diff. **Any historical divergence means the backfill didn't preserve identity — fix the backfill, never adjust the query.**
4. Only after a **full billing cycle of exact daily and monthly equality** for a cohort, switch the recorded value to *(remote + local workflow)* and remove the legacy local count — **in a single release**, never overlapped, or recipients get double-counted.
5. Same procedure for storage metering: the `emails` row count becomes a gRPC count. `CalculateStorageOverchargeCommand` already uses `GetGhostStorage` and needs only CRM attachment bytes folded in.

> A mismatch is a **release blocker**, not a warning.

## 5. Open questions

1. Who owns the invalid-domain credit counter once dispatch moves to email-api — email-api (which will own domain-auth state) or the monolith `Account` settings where it lives today?
2. Storage metering counts `emails` rows as storage units. Once `emails` lives in email-api, should that count come from `FeatureUsage.Show`, or should storage metering be **redefined in actual bytes** while we're touching it?
3. Are there accounts with known historical usage mismatches (see [[Usage mismatch]]) that would make the parallel-run equality gate impossible to satisfy? If so, reconcile those **before** the migration, not during.
