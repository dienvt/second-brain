---
tags: [comvex, digima, backend, domain, tenant, account, onboarding, runbook]
project: digima-backend-app
source: app/Jobs/Account/CreateAccountJob.php
created: 2026-04-23
---

# Onboarding a new Account

Runbook for provisioning a new tenant ([[Account]]) in `digima-backend-app`. Covers HTTP entry, the sync create job, per-account DB provisioning, post-create events, and the follow-up steps an operator must do to make the account usable.

> TL;DR: Only `CreateAccountJob` is synchronous. Microservice databases (Filter, Activity, Email, SMS, Line, etc.) and the Kafka event are provisioned **asynchronously via queued listeners** after `Account\CreatedEvent` fires. A created account has **no users, no active feature subscriptions, and no SendGrid** — operators must add those.

---

## 1. Entry points

### Primary: Admin API

`POST /admin/v2/accounts` — `app/Http/Controllers/Admin/v2/Account/AccountsController.php@store` (L99-128)

- Request class: `app/Http/Requests/Admin/Account/CreateAccountRequest.php`
- `authorize()` returns `true` — no policy guard beyond admin auth middleware
- Controller builds `CreateAccountJob` and dispatches it **synchronously** (`dispatch_sync`)
- Returns 201 with the transformed Account

### E2E testing

`php artisan digima-tests-e2e:create-account {suites} [--y] [--pretend]` — `app/Console/Commands/Tests/E2E/CreateAccountCommand.php`
- Wraps `CreateAccountJob` and additionally provisions microservice DBs for the given suite(s) (e.g. `email`, `sms`).

### Low-level DB commands (not a full onboarding)

| Command | Purpose |
|---|---|
| `digima:db-account-create {account}` | Create `dgm_account_{id}` DB only |
| `digima:db-account-migrate {account} [--force] [--pretend]` | Run per-account migrations |
| `digima:db-account-seed {account} [--class=...] [--dev] [--ignore=]` | Run per-account seeders |
| `digima:db-account-migrate-refresh {account}` | Rollback + migrate + seed |
| `digima:db-account-migrate-reset {account}` | Rollback all |
| `digima:db-account-migrate-rollback {account}` | Rollback last batch |

`{account}` supports `all`, a comma list (`1,2,3`), or a range (`1-3`).

---

## 2. Request fields

Defined in `CreateAccountRequest.php`.

**Required:**
- `name` (string ≤100, unique in core DB)
- `phone_number` (string, phone format, ≤50)
- `street`, `city`, `region`, `postal_code` (strings, ≤150 / ≤50)
- `industry_id` (exists in `core.industries`)

**Optional:**
- `website_url` (valid URL, ≤300)
- `status` (default `enabled` — **see next section, initially forced to `disabled`**)
- `cancellation_reasons[]` (required iff `status = canceled`)
- `is_demo` (bool, default `false`)
- `demo_expires_at` (required iff `is_demo = true`)
- `industry_classification_ids[]` (must belong to `industry_id`)
- `timezone_id` (default `Timezone::DEFAULT_TIMEZONE_ID` — Asia/Tokyo)
- `locale_id` (default `Locale::DEFAULT_LOCALE_ID` — ja_JP)
- `billing_properties[]`
- `sendgrid_settings[]` (keys: `api_key`, `email`, `ips`, `password`, `username`, `user_id`)

`country_id` is **not in the request** — it's hard-set to `Country::DEFAULT_COUNTRY_ID` by the controller (L113).

---

## 3. `CreateAccountJob` — step by step

File: `app/Jobs/Account/CreateAccountJob.php` (332 lines)

```
┌──────────────────────────────────────────────────────────────┐
│ handle()  (L179-209)                                         │
├──────────────────────────────────────────────────────────────┤
│ 1. makeAttributes()                      L181                │
│    - pulls default billing_properties from core Setting      │
│      KEY_DEFAULT_BILLING_PROPERTIES if not supplied (L245-49)│
│    - FORCES status = STATUS_DISABLED (L260)                  │
│    - strips pricing_plan keys (refineBillingProperties L286) │
│                                                              │
│ 2. new Account($attributes)              L183                │
│ 3. assignCode() + assignApiKey()         L185-186            │
│ 4. save() → persist to core DB           L188                │
│ 5. industryClassifications()->sync(...)  L190                │
│                                                              │
│ 6. setUpAccountDatabase()                L192                │
│    ├── skip if DB already exists         L312                │
│    └── buildDatabase($accountId):        L318                │
│        a) DatabaseManager::createAccountDatabase  (MySQL CREATE) │
│        b) DatabaseManager::migrateAccountDatabase (artisan)  │
│        c) DatabaseManager::seedAccountDatabase    (artisan)  │
│        ↳ on failure: deleteAccountDatabase + $account->delete│
│           + throw DatabaseException         L321-324         │
│                                                              │
│ 7. Apply requested status                L195-199            │
│    $account->status = $this->status                          │
│    $account->status_updated_at = now()                       │
│    $account->save()                                          │
│                                                              │
│ 8. fire(new CreatedEvent($account))      L201                │
│ 9. notify(Administrator::notifiables(),                      │
│           new AccountCreatedNotification)  L204              │
│ 10. log_info('Account created', ...)     L206                │
│ 11. return $account                      L208                │
└──────────────────────────────────────────────────────────────┘
```

### ⚠️ Two-phase status

There's a short window where the account row exists with `status = disabled` (L260) **before** the DB is provisioned. The requested `status` is applied only after `setUpAccountDatabase()` succeeds (L195). Any failure in DB provisioning **deletes both the DB and the Account row** (L321-322) and throws `DatabaseException`.

This is why `CreatedEvent` is fired after the final status save — consumers see the correct final status.

---

## 4. Per-account DB provisioning

Service: `app/Services/DatabaseManager.php`

### Naming

Pattern: **`dgm_account_{id}`** (plus `_test` suffix in testing). Defined by:
- `DATABASE_NAME_PREFIX = 'dgm'` (L21)
- `DATABASE_NAME_SEPARATOR = '_'` (L22)
- `CONNECTION_NAME_ACCOUNT = 'account'` (L16)

Also registered: `job` (MongoDB, async jobs), `statistic` (MongoDB, analytics).

### Migrations

- Path: `database/migrations/account/mysql/` (197+ files) and `database/migrations/account/mongodb/`
- Executed via `digima:db-account-migrate {id} --force`
- Tables created: `settings`, `users`, `devices`, `feature_subscriptions`, contacts/companies/workflows/emails/calls/web/… (200+ tables).

### Seeders (production default)

Entry: `database/seeders/ProductionAccountSeeder.php`

| Seeder | Seeds |
|---|---|
| `AccountSettingSeeder` | Default settings (`contact_*_order`, `invoice_notification_email_addresses`, empty `login_ip_whitelist`, etc.) |
| `AccountContactStatusSeeder` | 60+ default contact statuses (Japanese real-estate sales stages) |
| `AccountCallOutcomeSeeder` | Default call outcome types |
| `AccountCustomerTypeSeeder` | Default customer classifications |
| `AccountLeadAcquisitionChannelSeeder` | Lead source channels (Web form, Phone, …) |

Pass `--dev` on the seed command to run `DevelopmentAccountSeeder` instead (adds test data).

---

## 5. `CreatedEvent` fan-out

Wired in `app/Providers/EventServiceProvider.php` (~L187-190).

| Listener / Subscriber | Sync/Async | What it does |
|---|---|---|
| `AttachFeatureFlagsToAccountSubscriber` | Sync | Attaches **all `is_published = true`** flags, then explicitly attaches `FEATURE_FLAG_CK_EDITOR_5` (see note below). Fires `Feature\Flag\AttachedEvent` per flag. |
| `InitializeFilterDatabaseListener` | **Queued** | gRPC → Filter microservice: `FilterClient\Handlers\Databases($account)->create()` |
| `InitializeActivityDatabaseListener` | **Queued** | gRPC → Activity microservice: `ActivityClient\Handlers\Databases($account)->create()` |
| `UpdateStatisticSubscriber` | Sync | Runs artisan statistics refresh |
| `ProduceAccountEventsSubscriber` | **Queued** | Publishes Kafka `tenant.backend_app.accounts.created.v1` (`AccountCreatedEventV1`) |
| `AccountCreatedNotification` (mail) | **Queued** | Notifies admins |

### Feature flags on create

`app/Listeners/Feature/Flag/AttachFeatureFlagsToAccountSubscriber.php` — L47-74:
- Attaches every `FeatureFlag` with `is_published = true` via pivot `syncWithoutDetaching`.
- Fires `Feature\Flag\AttachedEvent($featureFlag, $account)` for each newly attached flag.
- **Extra special-case at L64-74**: always attaches `FEATURE_FLAG_CK_EDITOR_5` (with a `FIXME` comment to remove once CKEditor5 is published).

### Feature subscriptions on create

**None.** Subscriptions for `email`, `sms`, `line`, `workflow`, `andpad`, etc. must be started separately (section 8, step 4). The microservice DBs for those features are provisioned by listeners on `Feature\Subscription\StartedEvent`, not on `CreatedEvent`.

---

## 6. External side effects at create time

| System | When | How |
|---|---|---|
| MySQL — core DB | Sync | Account row, industry_classifications pivot |
| MySQL — per-account DB (`dgm_account_{id}`) | Sync | create + migrate + seed |
| MongoDB — per-account `job`, `statistic` | On first use | via DatabaseManager connections |
| Filter microservice | Async (queue) | gRPC via `FilterClient` |
| Activity microservice | Async (queue) | gRPC via `ActivityClient` |
| Kafka | Async (queue) | `tenant.backend_app.accounts.created.v1` |
| Admin email | Async (queue) | `AccountCreatedNotification` |

Not touched on create:
- Email / SMS / Line / Andpad / LeadAcquisition DBs → provisioned on their own `Feature\Subscription\StartedEvent`.
- SendGrid → only if `sendgrid_settings` was supplied in the request (stored as JSON, no external call).

---

## 7. Sequence diagram

```
Admin                AccountsController       CreateAccountJob          DatabaseManager         Event bus             Listeners (queued)
  │                          │                         │                        │                       │                       │
  │── POST /admin/v2/accounts┤                         │                        │                       │                       │
  │                          │── new CreateAccountJob ─▶                        │                       │                       │
  │                          │── dispatch_sync ────────▶ handle()               │                       │                       │
  │                          │                         │── makeAttributes       │                       │                       │
  │                          │                         │  (status=DISABLED)     │                       │                       │
  │                          │                         │── save Account ────────▶ core DB               │                       │
  │                          │                         │── sync industry_class. ▶ core DB               │                       │
  │                          │                         │── setUpAccountDatabase ▶ create dgm_account_N  │                       │
  │                          │                         │                        │  migrate              │                       │
  │                          │                         │                        │  seed                 │                       │
  │                          │                         │── status = requested ──▶ core DB               │                       │
  │                          │                         │── fire CreatedEvent ───┼──────────────────────▶│                       │
  │                          │                         │                        │                       ├─▶ AttachFeatureFlags (sync)
  │                          │                         │                        │                       ├─▶ UpdateStatisticSub (sync)
  │                          │                         │                        │                       ├─▶ InitializeFilterDB ──▶ queue ─▶ gRPC FilterClient
  │                          │                         │                        │                       ├─▶ InitializeActivityDB▶ queue ─▶ gRPC ActivityClient
  │                          │                         │                        │                       └─▶ ProduceAccountEvents▶ queue ─▶ Kafka
  │                          │                         │── notify admins ───────┼──────────────────────────────────────────────▶ queue ─▶ mail
  │                          │                         │── log_info             │                       │                       │
  │◀─ 201 + Account ─────────┤◀── return Account ──────┤                        │                       │                       │
```

---

## 8. Post-onboarding checklist

A freshly created account has: correct status, unique `code` + `api_key`, provisioned per-account DB with default settings/contact-statuses/call-outcomes, all published feature flags attached — **and nothing else**. Operators must:

### Step 1 — Create the owner user
```
POST /admin/v2/accounts/{account_id}/users
{
  "role": "owner",
  "email": "owner@company.com",
  "first_name": "…",
  "last_name": "…",
  "is_enabled": true
}
```
Controller: `app/Http/Controllers/Admin/v2/Account/User/UsersController.php@store`. User is created with `activation_token` set and `activated_at = null`.

### Step 2 — Owner activates
Owner follows the activation link from email (sets password etc.). Until this runs, the account has no one who can log in.

### Step 3 — Start feature subscriptions
```
POST /admin/v2/accounts/{account_id}/features/subscriptions/{featureName}/start
```
Each subscription fires `Feature\Subscription\StartedEvent`, which triggers microservice DB initialization listeners for Email, SMS, Line, Andpad, LeadAcquisition, etc.

Start only what the contract covers. See [[Account]] § Feature flag gating for flag-vs-subscription semantics.

### Step 4 — Configure SendGrid (if email feature is in use)
If not passed at create time:
```
PUT /admin/v2/accounts/{account_id}
{
  "sendgrid_settings": { "api_key": "…", "email": "…", "username": "…", "password": "…", "user_id": "…", "ips": "…" },
  "update_sendgrid_ips": true
}
```
The `update_sendgrid_ips` special action triggers IP propagation on the SendGrid sub-user.

### Step 5 — IP whitelist (if `IP_RESTRICTIONS` feature is attached)
```
PUT /admin/v2/accounts/{account_id}/settings
{
  "key": "login_ip_whitelist",
  "value": [{ "ip": "203.0.113.0" }, { "ip": "203.0.113.1" }]
}
```
Enforced by `Account::authorizesIp($ip)` in middleware.

### Step 6 — Billing properties / pricing plan
Billing props keyed on `pricing_plan` are **stripped at create time** (`refineBillingProperties` L286). Set them after the fact:
```
PUT /admin/v2/accounts/{account_id}
{
  "billing_properties": { "contacts": {…}, "pricing_plan": "pricing_plan_…", … },
  "pricing_plan": "pricing_plan_…",
  "max_workflow_count": 100
}
```

### Step 7 — Additional users
Same endpoint as step 1. Roles: `owner`, `admin`, `user`.

### Step 8 — Franchise relationships (optional)
Requires `FRANCHISE_HEADQUARTERS_MANAGEMENT` subscription active. See [[FranchiseAccount]].
```
POST /admin/v2/accounts/{franchisor_id}/franchisee-accounts
{ "add_franchisee_accounts": [101, 102] }
```

### Step 9 — Flip to ENABLED (if created as disabled)
```
PUT /admin/v2/accounts/{account_id}
{ "status": "enabled" }
```

### Step 10 — Seed dev data (dev/staging only)
```
php artisan digima:db-account-seed {account_id} --dev
```

---

## 9. Failure handling

| Failure point | Behavior |
|---|---|
| Request validation fails | 422 before job runs |
| MySQL cannot create `dgm_account_{id}` | `buildDatabase` returns false → `DatabaseManager::deleteAccountDatabase` + `$account->delete()` + `throw DatabaseException` |
| Migration fails (non-zero exit) | Same rollback path |
| Seeder fails (non-zero exit) | Same rollback path |
| Any listener on `CreatedEvent` fails (queue) | Account and DB remain; queued jobs retry per queue config. Filter/Activity DB may be in inconsistent state → check queue failures |
| Admin notification fails | Doesn't affect account creation (queued) |

Rollback is best-effort on the synchronous path only. If a queued listener (e.g. Filter DB init) fails permanently, manual remediation is needed — no compensating deletion is triggered.

---

## 10. Quick file map

| Concern | Path |
|---|---|
| HTTP controller | `app/Http/Controllers/Admin/v2/Account/AccountsController.php` |
| Request validation | `app/Http/Requests/Admin/Account/CreateAccountRequest.php` |
| Create job | `app/Jobs/Account/CreateAccountJob.php` |
| DB manager | `app/Services/DatabaseManager.php` |
| Core model | `app/Models/Core/Account.php` |
| `CreatedEvent` | `app/Events/Account/CreatedEvent.php` |
| Feature flag subscriber | `app/Listeners/Feature/Flag/AttachFeatureFlagsToAccountSubscriber.php` |
| Filter DB init listener | `app/Listeners/Account/InitializeFilterDatabaseListener.php` |
| Activity DB init listener | `app/Listeners/Account/InitializeActivityDatabaseListener.php` |
| Kafka producer | `app/Services/EventProducer/Handlers/Accounts.php` |
| Per-account migrations | `database/migrations/account/mysql/` |
| Per-account seeders | `database/seeders/ProductionAccountSeeder.php` + `Account*Seeder.php` |
| DB artisan commands | `app/Console/Commands/Database/Account/` |
| E2E helper | `app/Console/Commands/Tests/E2E/CreateAccountCommand.php` |

---

## Related

- [[Account]] — tenant model reference
- [[FranchiseAccount]] — HQ ↔ branch relationships
