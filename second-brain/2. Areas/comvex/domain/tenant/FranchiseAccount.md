---
tags: [comvex, digima, backend, domain, tenant, franchise]
project: digima-backend-app
source: app/Models/Core/FranchiseAccount.php
created: 2026-04-23
---

# Franchise Accounts

A **FranchiseAccount** is a hierarchical HQ → branch relationship between two [[Account]] tenants. A **franchisor** (parent HQ) manages multiple **franchisees** (child branches) centrally.

Key property: **no config inheritance**. Each franchisee keeps its own isolated DB, users, settings, and feature subscriptions. Only explicit jobs sync data (e.g. contacts). This is a **referential** relationship, not a nesting relationship.

---

## 1. Model

`app/Models/Core/FranchiseAccount.php` (~60 lines)

- Core DB, table `franchise_accounts`.
- Columns: `id`, `franchisor_id` (FK → `accounts.id`), `franchisee_id` (FK → `accounts.id`), `created_at`.
- **No `updated_at`** — records are create-and-delete only.

Relations:
- `franchiseeAccount()` → belongsTo `Account` via `franchisee_id`
- `franchisorAccount()` → belongsTo `Account` via `franchisor_id`

From the Account side (`Account.php`):
- `franchiseeAccounts()` — this account as franchisor (has many children)
- `franchisorAccounts()` — this account as franchisee (has many parents)

---

## 2. Hierarchy rules

- **Only one level of direct relationship.** A franchisee cannot itself have franchisees (no sub-franchises).
- An account **can be both** — franchisee of one HQ and franchisor of another — but cycles are rejected.
- **Cycle prevention**: `canAddFranchisee()` in `FranchiseAccountsController` rejects:
  - Self-reference (A cannot be franchisee of A)
  - Recursive cycles (A → B → C → A)

---

## 3. Lifecycle

### Create
**Endpoint**: `POST /admin/v2/accounts/{account}/franchisee-accounts`
**Controller**: `app/Http/Controllers/Admin/v2/Account/Franchise/FranchiseAccountsController.php@store` (~76 lines)

Body:
```json
{ "add_franchisee_accounts": [101, 102, 103] }
```

Flow:
1. `canAddFranchisee()` cycle check.
2. For each franchisee ID, dispatch `CreateFranchiseAccountJob` **synchronously**.
3. Job (`app/Jobs/Account/FranchiseAccount/CreateFranchiseAccountJob.php`, ~51 lines): persists pivot row + logs.
4. Clear caches for franchisor and all affected franchisees.
5. Return transformed rows (HTTP 201).

### List
**Endpoint**: `GET /admin/v2/accounts/{account}/franchisee-accounts`
Query: `order_col` (default `created_at`), `order_dir` (default `desc`), `per_page` (default 25).

### Delete
**Endpoint**: `DELETE /admin/v2/accounts/{account}/franchisee-accounts/{franchiseeAccountId}`
- Looks up row by `(franchisor_id, franchisee_id)`; 404 if missing.
- Dispatches `DeleteFranchiseAccountJob` (`app/Jobs/FranchiseAccount/DeleteFranchiseAccountJob.php`, ~35 lines).
- Clears caches.

### Transformer output
`FranchiseAccountTransformer`:
```json
{
  "id": 1,
  "franchisee_id": 101,
  "franchisor_id": 100,
  "created_at": "2025-01-15T10:00:00Z"
}
```
Optional includes: `franchisee_account`, `franchisor_account` (full Account objects).

---

## 4. Automatic cleanup

`app/Listeners/FranchiseAccount/DeleteFranchiseAccountsSubscriber.php` (~78 lines) wipes franchise rows automatically when triggered by these events:

| Event | Rule | Line |
|---|---|---|
| `Account\DeletedEvent` | Delete all rows where account is franchisor **or** franchisee | L25 |
| `Account\DisabledEvent` | Same as above | L26 |
| `Account\CancelledEvent` | Same as above | L27 |
| `Feature\Subscription\EndedEvent` (`FRANCHISE_HEADQUARTERS_MANAGEMENT`) | Delete only rows where account is **franchisor** | L54-77 |

Query pattern:
```php
FranchiseAccount::query()
  ->where('franchisee_id', $account->id)
  ->orWhere('franchisor_id', $account->id)
  ->each(fn($fa) => dispatch_sync(new DeleteFranchiseAccountJob($fa)));
```

---

## 5. Feature-flag gating

**`FRANCHISE_HEADQUARTERS_MANAGEMENT`** is the prerequisite for franchise management.

- While active: `AccountTransformer` exposes `franchisee_accounts` / `franchisor_accounts` includes (cached).
- While inactive: transformer returns `null` for those includes (`AccountTransformer.php:182-185`).
- When the subscription ends: cleanup subscriber deletes all franchise rows for this franchisor.

---

## 6. Caching

Defined on `Account.php:741-779`:

| Key | TTL |
|---|---|
| `account:{franchisorAccountId}-franchisee-accounts` | 300s |
| `account:{franchiseeAccountId}-franchisor-accounts` | 300s |

Helpers: `franchiseeAccountsCached()`, `franchisorAccountsCached()`, `clearFranchiseAccountsCache($franchisorId, $franchiseeIds)`.

---

## 7. Contact synchronization jobs

Beyond the pivot lifecycle, `app/Jobs/FranchiseAccount/` contains sync jobs. These are triggered by contact-side business logic, not the FranchiseAccount model itself.

| Job | Purpose |
|---|---|
| `SendContactsToFranchiseeAccountJob` | Push contacts from HQ to branches |
| `ReceiveContactFromFranchisorAccountJob` | Franchisee-side ingest |
| `ReceiveContactCreatedStatusFromFranchiseeAccountJob` | Ack back to HQ |

---

## 8. File map

| Concern | Path |
|---|---|
| Pivot model | `app/Models/Core/FranchiseAccount.php` |
| Account relations | `app/Models/Core/Account.php` (`franchiseeAccounts`, `franchisorAccounts`, cache helpers) |
| Admin API | `app/Http/Controllers/Admin/v2/Account/Franchise/FranchiseAccountsController.php` |
| Create job | `app/Jobs/Account/FranchiseAccount/CreateFranchiseAccountJob.php` |
| Delete job | `app/Jobs/FranchiseAccount/DeleteFranchiseAccountJob.php` |
| Contact sync jobs | `app/Jobs/FranchiseAccount/*` |
| Cleanup subscriber | `app/Listeners/FranchiseAccount/DeleteFranchiseAccountsSubscriber.php` |
| Transformer | `app/Transformers/Core/FranchiseAccountTransformer.php` |
| Request validation | `app/Http/Requests/Admin/Account/FranchiseAccount/{Create,List}FranchiseAccountsRequest.php` |

---

## Related

- [[Account]] — the tenant model.
