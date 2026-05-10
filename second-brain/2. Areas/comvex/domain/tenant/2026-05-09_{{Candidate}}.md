---
tags: [comvex, digima, backend, domain, tenant, index]
project: digima-backend-app
created: 2026-04-23
---

# Tenant Domain

Multi-tenancy foundation of `digima-backend-app`. Every organization on the platform is an **Account** with its own isolated database, users, billing, and feature flags. Accounts can be linked in **franchise** (HQ → branch) relationships.

## Notes

- [[Account]] — tenant model: structure, REST APIs (admin v2 / user v2 / internal v3 / sms forwarder v1), lifecycle events, feature-flag gating, caching.
- [[FranchiseAccount]] — franchisor/franchisee pivot: APIs, automatic cleanup, feature gating (`FRANCHISE_HEADQUARTERS_MANAGEMENT`), contact sync jobs.

## Runbooks

- [[Onboarding a new Account]] — end-to-end provisioning: HTTP entry → `CreateAccountJob` → per-account DB → `CreatedEvent` fan-out → post-onboarding checklist.

## High-level map

```
Core DB
├── accounts ............... tenant root (Account)
├── franchise_accounts ..... HQ ↔ branch pivot (FranchiseAccount)
├── user_references ........ cross-tenant user counters
├── feature_flags .......... global catalog
└── account_feature_flag_pivot

Per-account DB (one per tenant)
├── users / devices / settings
├── contacts / companies / groups
├── emails / calls / line / sms
├── workflows / imports / exports
└── activities / notifications / reminders
```

## Lifecycle at a glance

```
create → enabled ──update──▶ enabled
              │
              ├──disable──▶ disabled  (franchise cleanup, usage recalculated)
              │
              └──cancel ──▶ canceled  (franchise cleanup, usage recalculated, Kafka produced)

delete (demo + expired only) → deleted  (franchise cleanup)
```

## Produced Kafka topics

- `tenant.backend_app.accounts.created.v1`
- `tenant.backend_app.accounts.updated.v1`
- `tenant.backend_app.accounts.cancelled.v1`

Handler: `app/Services/EventProducer/Handlers/Accounts.php`.
