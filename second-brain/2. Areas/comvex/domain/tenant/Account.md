---
tags: [comvex, digima, backend, domain, tenant, account]
project: digima-backend-app
source: app/Models/Core/Account.php
created: 2026-04-23
---

# Account Domain

The **Account** domain is the tenant / multi-tenancy foundation of Digima. Every Account is one organization (a tenant) using the platform, with its own isolated database, configuration, users, billing, and feature flags.

Repo: `digima-backend-app` (Laravel / PHP monolith).

---

## 1. Core concept

- An **Account** = one tenant (a company / organization using Digima).
- Each Account lives in the **core DB** (`accounts` table) but has its **own dedicated per-account database** for domain data (users, contacts, emails, calls, workflows, imports, exports, etc.).
- `DatabaseManager` handles DB provisioning: creation, migration, seeding, and connection switching by account context.

### Two-database pattern

| DB | Contents |
|---|---|
| **Core DB** | `accounts`, `franchise_accounts`, `user_references`, `feature_flags`, all pivot tables |
| **Per-account DB** (one per tenant) | users, contacts, emails, calls, workflows, imports, exports, settings, notifications, ... |

---

## 2. Key models

### `Account` — `app/Models/Core/Account.php` (~917 lines)

| Group | Fields |
|---|---|
| Identity | `id`, `code` (10-char unique), `api_key` (60-char) |
| Name / contact | `name`, `website_url`, `phone_number` |
| Address | `street`, `city`, `region`, `postal_code` |
| Lifecycle | `status` (`enabled` / `disabled` / `canceled`), `status_updated_at`, `cancellation_reasons[]` |
| Flags | `is_demo`, `demo_expires_at`, `is_sandbox`, `initial_version` |
| Billing | `billing_properties` (array, ~70 constants: pricing plans, package tiers, discounts, per-feature caps for contacts / SMS / Email / LINE / calls / workflows / storage) |
| Integrations | `sendgrid_settings`, `pricing_plan`, `max_workflow_count` |
| FKs | `industry_id`, `country_id`, `timezone_id`, `locale_id` |
| Counters | `users_count`, `user_references_count`, `activated_user_references_count`, `disabled_user_references_count` |

**Helpers**: `assignCode()`, `assignApiKey()`, `isEnabled/Disabled/Canceled()`, `isDemoExpired()`, `authorizesIp($ip)` (IP whitelist, when `IP_RESTRICTIONS` feature active), `owner()`, `billingPropertiesForFeature($feature)`, `getSendgridApiKey()`, `getInvoiceNotificationEmailAddresses()`.

**Relations**:
- `users()`, `devices()`, `settings()` — from the account DB.
- `featureFlags()` — belongsToMany via pivot `account_feature_flag_pivot` (core DB).
- `industryClassifications()` — pivot `account_industry_classification_pivot`.
- `country()`, `industry()`, `timezone()`, `locale()` — belongsTo.
- `userReferences()` — hasMany in core DB (cross-account counters).
- `franchiseeAccounts()` / `franchisorAccounts()` — hasMany `FranchiseAccount`.

**Lookups**: `Account::findByCode(string)`, `Account::findByApiKey(string)`.

**Static catalogs**: `getBillingPropertyTypes()`, `getPricingPlanBillingProperties()`, `getCancellationReasons()`, `getStatuses()`.

**Status values**: `STATUS_ENABLED`, `STATUS_DISABLED`, `STATUS_CANCELED`.

**Cancellation reasons (14)**: `CHANGE_OF_MANAGEMENT`, `CONTRACT_ENDED`, `DIFFERENT_FROM_EXPECTATION`, `DIFFICULT_TO_USE`, `LACK_OF_FINANCIAL_RESOURCES`, `LOSS_KEY_USER`, `MANY_BUGS`, `MERGED_TO_ANOTHER_ACCOUNT`, `NOT_ENOUGH_FEATURES`, `OTHERS`, `POOR_ADOPTION`, `POOR_CUSTOMER_SERVICE`, `PRODUCT_WAS_NOT_GOOD_FIT`, `SWITCHED_TO_COMPETITOR`.

### `FranchiseAccount` — `app/Models/Core/FranchiseAccount.php`

Minimal pivot: `id`, `franchisor_id`, `franchisee_id`, `created_at` (no `updated_at` — create/delete only). See [[FranchiseAccount]].

### `Setting` — `app/Models/Account/Setting.php`

Per-account key/JSON-value settings. Notable keys: `KEY_LOGIN_IP_WHITELIST`, `KEY_INVOICE_NOTIFICATION_EMAIL_ADDRESSES`, `KEY_CONTACT_CREATION_ATTRIBUTES`, `KEY_CONTACT_DISPLAY_NAME_ORDER`, `KEY_CONTACT_PROFILE_ATTRIBUTES_ORDER`, `KEY_CONTACT_TABLE_COLUMNS_ORDER`, `KEY_WEB_TRACKING`, `KEY_IMPORT`, `KEY_BULK_EMAIL`, `KEY_BULK_ACTION`, `KEY_EXPORT`.

### `User` — `app/Models/Account/User.php`

Lives in the account DB. `UserReference` (`app/Models/Core/UserReference.php`) is the core-DB mirror used for cross-tenant counters.

---

## 3. Exposed functionalities

### Admin API v2 — `routes/admin/v2/administrator.php`

**Accounts** (`app/Http/Controllers/Admin/v2/Account/AccountsController.php`, 174 lines):

| Method | Path | Purpose |
|---|---|---|
| GET | `/accounts` | List with filters: `search`, `status`, `is_demo`, `industry_id`, `country_id`, `timezone_id`, `locale_id`, `created_after/before`, sort, paginate |
| GET | `/accounts/{account}` | Show one |
| POST | `/accounts` | Create → persist, assign `code` + `api_key`, provision per-account DB (migrate + seed), fire `CreatedEvent`. 201 |
| PUT | `/accounts/{account}` | Update; special actions `generate_new_api_key`, `update_sendgrid_ips`; sync feature flags; fire `Updated` / `Disabled` / `Cancelled` event |
| DELETE | `/accounts/{account}` | **Only expired demo accounts** may be deleted; wipes users + account; fire `DeletedEvent` |

**Franchises** (`app/Http/Controllers/Admin/v2/Account/Franchise/FranchiseAccountsController.php`, 128 lines):

| Method | Path |
|---|---|
| GET | `/accounts/{account}/franchisee-accounts` |
| POST | `/accounts/{account}/franchisee-accounts` (body: `add_franchisee_accounts[]`, cycle-check) |
| DELETE | `/accounts/{account}/franchisee-accounts/{franchiseeAccountId}` |

**Other account-scoped routes**:
- Feature flags: `GET/PUT /accounts/{account}/features/flags`
- Feature subscriptions: `index` / `show` / `start` / `end` / `update` on `/accounts/{account}/features/subscriptions[/{featureName}]`
- Feature usage: `GET /accounts/{account}/features/usage`, `POST .../recalculate`
- Billing: invoices `index/show/update/destroy/download`; segments `store/destroy`
- Settings: `GET/PUT /accounts/{account}/settings`, `GET /accounts/{account}/settings/{setting}`
- Users: full CRUD under `/accounts/{account}/users[/{userId}]` + `PUT .../reset-activation`

### User API v2 — `routes/user/v2/user.php`

`app/Http/Controllers/User/v2/AccountController.php` (57 lines):
- `GET /account` — caller's own account (policy-guarded).
- `PUT /account` — self-update (name, website_url, phone, address, industry/timezone/locale; can `generate_new_api_key`).

### Internal API v3 — `routes/internal/v3/internal.php`

`app/Http/Controllers/Internal/v3/AccountsController.php` (72 lines):
- `GET /accounts` — admin-v2 filters plus `ids[]`; **omits `billing_properties` column** for perf.
- `GET /accounts/{account}`.

### SMS Forwarder v1 — `routes/sms/forwarder/v1/forwarder.php`

- `GET /account` — for the SMS forwarder integration.

### Request validation classes

- `app/Http/Requests/Admin/Account/{Create,Update,List}AccountRequest.php`
- `app/Http/Requests/Admin/Account/FranchiseAccount/{Create,List}FranchiseAccountsRequest.php`
- `app/Http/Requests/User/Account/UpdateAccountRequest.php`
- `app/Http/Requests/Internal/Accounts/ListAccountsRequest.php`

### Response shape — `app/Transformers/Core/AccountTransformer.php`

Base payload = all scalar account columns + counters. Optional `?include=`:

- `industry`, `industry_classifications`, `country`, `locale`, `timezone`
- `devices`, `settings`, `users` (supports `me_first`)
- `feature_subscriptions`, `feature_flags` (pageable, default 100)
- `franchisee_accounts`, `franchisor_accounts` — **cached**, and returned only when `FRANCHISE_HEADQUARTERS_MANAGEMENT` feature is active (`AccountTransformer.php:182-185`).

---

## 4. Emitted events

All lifecycle changes flow through jobs, which fire domain events. `ProduceAccountEventsSubscriber` then republishes selected events to Kafka via protobuf.

### Lifecycle events

| Event | Fired at | Kafka topic | Notes |
|---|---|---|---|
| `Account\CreatedEvent` | `CreateAccountJob.php:201` | `tenant.backend_app.accounts.created.v1` (`AccountCreatedEventV1`) | After persist + DB provision |
| `Account\UpdatedEvent` | `UpdateAccountJob.php:151` | `tenant.backend_app.accounts.updated.v1` (`AccountUpdatedEventV1`) | Not fired on transitions to disabled/canceled |
| `Account\DisabledEvent` | `UpdateAccountJob.php:243` | *(not produced)* | Runs `CalculateUsageJob`; triggers franchise cleanup |
| `Account\CancelledEvent` | `UpdateAccountJob.php:248` | `tenant.backend_app.accounts.cancelled.v1` (`AccountCancelledEventV1`) | Runs `CalculateUsageJob`; triggers franchise cleanup |
| `Account\DeletedEvent` | `DeleteAccountJob.php:41` | *(not produced)* | Triggers franchise cleanup |
| `Account\Setting\UpdatedEvent` | SettingObserver | *(not produced)* | Payload: `getSetting(): Setting` |

Payload on all lifecycle events: `getAccount(): Account`.

### Subscribers / listeners (`EventServiceProvider` ~L187-190)

On `CreatedEvent`:
- `InitializeFilterDatabaseListener` (queued) — filter DB.
- `InitializeActivityDatabaseListener` (queued) — activity DB via ActivityClient.
- `UpdateStatisticSubscriber` — artisan stats refresh.
- `AttachFeatureFlagsToAccountSubscriber` — attach all published flags.
- `ProduceAccountEventsSubscriber` (queued).

On `Disabled` / `Cancelled` / `Deleted`:
- `DeleteFranchiseAccountsSubscriber` — removes franchise rows. See [[FranchiseAccount]].

### Feature flag events (tied to accounts)

| Event | Fired at | Payload |
|---|---|---|
| `Feature\Flag\AttachedEvent` | `UpdateAccountJob.php:322-327` | `(FeatureFlag, Account)`. If flag is `FILE_API`, initializes file DB. |
| `Feature\Flag\DetachedEvent` | `UpdateAccountJob.php:284-291` | `(FeatureFlag, Account)` |

### Feature subscription events (cross-cutting)

- `Feature\Subscription\StartedEvent` — triggers DB init for SMS, Email, Line, Andpad, etc.
- `Feature\Subscription\EndedEvent` — if `FRANCHISE_HEADQUARTERS_MANAGEMENT` ends, the subscriber removes all franchise rows where this account is franchisor.

### Event producer

`app/Services/EventProducer/Handlers/Accounts.php` — protobuf serialization for the three produced topics. All publishing is queued.

Common produced fields: `account_id`, `account_name`, `status`, `timezone`, `created_at`, `status_updated_at` (+ `occurred_at` on Cancelled).

---

## 5. Feature flag gating

- **`FRANCHISE_HEADQUARTERS_MANAGEMENT`** — required to manage franchisees. Transformer hides franchise includes when inactive. On subscription end, all franchise rows for this franchisor are deleted.
- **`IP_RESTRICTIONS`** — activates `authorizesIp($ip)` based on `KEY_LOGIN_IP_WHITELIST` setting.
- **`FILE_API`** — on attach, initializes file DB.

Feature-flag cache key: `account:{id}.model:feature_flag:{code}` (1 day).

---

## 6. Caching

| Key | TTL | Source |
|---|---|---|
| `account:{id}.model:feature_flag:{code}` | 1 day | feature flag checks |
| `account:{franchisorAccountId}-franchisee-accounts` | 300s | franchise list |
| `account:{franchiseeAccountId}-franchisor-accounts` | 300s | reverse franchise list |

Helpers at `Account.php:741-779`: `franchiseeAccountsCached()`, `franchisorAccountsCached()`, `clearFranchiseAccountsCache($franchisorId, $franchiseeIds)`.

---

## 7. Quick file map

| Concern | Path |
|---|---|
| Tenant model | `app/Models/Core/Account.php` |
| Per-account settings | `app/Models/Account/Setting.php` |
| Franchise pivot | `app/Models/Core/FranchiseAccount.php` |
| Admin CRUD | `app/Http/Controllers/Admin/v2/Account/AccountsController.php` |
| Franchise CRUD | `app/Http/Controllers/Admin/v2/Account/Franchise/FranchiseAccountsController.php` |
| Self-service | `app/Http/Controllers/User/v2/AccountController.php` |
| Internal | `app/Http/Controllers/Internal/v3/AccountsController.php` |
| Create/update/delete jobs | `app/Jobs/Account/{CreateAccountJob,UpdateAccountJob,DeleteAccountJob}.php` |
| Lifecycle events | `app/Events/Account/{Created,Updated,Disabled,Cancelled,Deleted}Event.php` |
| Setting event | `app/Events/Account/Setting/UpdatedEvent.php` |
| Flag events | `app/Events/Feature/Flag/{Attached,Detached}Event.php` |
| Kafka producer | `app/Services/EventProducer/Handlers/Accounts.php` |
| Franchise cleanup | `app/Listeners/FranchiseAccount/DeleteFranchiseAccountsSubscriber.php` |
| Transformer | `app/Transformers/Core/AccountTransformer.php` |

---

## Related

- [[FranchiseAccount]] — HQ → branch relationships between accounts.
