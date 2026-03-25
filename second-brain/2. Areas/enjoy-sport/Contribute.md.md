# Contributing to api_ticket

Thank you for contributing!

This project is a Go-based ticket/booking API built with **Clean Architecture** + **Domain-Driven Design (DDD)** principles, following pragmatic Go patterns commonly used in production systems (2025–2026 style).

The goal is to keep the domain pure, dependencies pointing inward, and the code maintainable, testable, and scalable.

## Core Architectural Principles

1. **Dependency Rule (Clean Architecture)**  
   - `domain` → nothing (only stdlib + shared kernel)  
   - `application` → only `domain` (including repository **interfaces**)  
   - `infrastructure` & `interfaces` → `application` + `domain`  
   → Never import outer layers from inner layers  
   → Application layer **can and must** import repository **interfaces** (ports), but **never** concrete implementations

2. **Project Structure Overview**
```
internal/
├── domain/                        # Pure business – no DB, no HTTP, no externals
│   ├── order/                     # Bounded context
│   │   ├── order.go               # Aggregate Root + Entity (usually one file)
│   │   ├── repository/            # Interfaces only (ports)
│   │   └── ...                    # value objects, errors, events
│   ├── payment/
│   ├── ticket/
│   └── shared/
│       └── vo/                    # Value Objects: OrderID, PaymentID, Money, ...
│
├── application/                   # Use cases / orchestration
│   ├── order/
│   │   ├── command/               # Write handlers (CQRS style)
│   │   ├── query/                 # Read handlers + DTOs
│   │   └── service.go             # (optional – if not full CQRS)
│   └── payment/
│
├── infrastructure/                # Adapters (outgoing)
│   └── persistence/
│       ├── order/
│       │   ├── order_model.go     # GORM structs
│       │   └── repository.go      # Concrete impl of domain repository interface
│       └── migrations/
│
└── interfaces/                    # Inbound adapters
└── http/
├── handler/
└── dto/
```

3. **Aggregates, Entities & Roots**

- Most aggregates use **one struct** that is **both Entity and Aggregate Root** (e.g. `Order`, `Payment`)  
- No separate `entity/` and `aggregate/` folders unless the aggregate is very complex  
- Child objects (OrderLine, PaymentAttempt…) are **Entities** but **not** Aggregate Roots  
- External code **only** holds references to Aggregate Roots  
- Reference other aggregates **only by ID** (never direct object references)

4. **IDs & Value Objects**

- Use **typed value objects** in `domain/shared/vo/` (e.g. `OrderID`, `PaymentID`, `Money`)  
- Current choice: **auto-increment BIGINT** as internal PK (fast, small, readable)  
  → Domain ID type: `type OrderID uint64`  
  → Exposed/public IDs: consider adding `public_id UUID` later if needed (hybrid pattern)  
- Money: immutable struct with `int64` amount (smallest unit) + `Currency`

5. **Write vs Read Paths (CQRS-lite)**

- **Writes / Commands**: Load full aggregate → call behavior → save aggregate  
  → Enforce all invariants  
- **Reads / Queries**: **Do NOT** always load full aggregate  
  → Use thin DTOs, dedicated query methods, or separate query repositories  
  → Avoid loading large collections (e.g. OrderLines) when only status/summary is needed

6. **Repository Pattern**

- **Interfaces** (ports) live in `domain/<context>/repository/`  
- **Concrete implementations** live in `infrastructure/persistence/<context>/`  
- Application layer depends **only** on interfaces → easy to mock & swap storage  
- Always load/save through the **Aggregate Root** repository

7. **Persistence**

- Domain aggregates ≠ GORM models  
- Use separate persistence structs (`OrderModel`, `PaymentModel`)  
- Map domain ↔ persistence in repository implementations  
- Use optimistic locking (`version` field)

8. **Naming & Conventions**

- Keep bounded contexts flat when simple (`order.go` instead of `aggregate/order.go`)  
- Use constructor functions: `NewOrder(...)`, `MustNewMoney(...)`  
- Domain behavior methods: `order.ApplyPayment(...)`, `order.CanCreateNewPayment()`  
- Thin layers:  
  - Handlers → map + call application  
  - Application → orchestrate (load → behavior → save)  
  - Domain → pure business rules & invariants

## Before Submitting a PR

- Business rules and invariants belong **only** in domain  
- Application services/handlers are orchestrators — no business logic  
- HTTP/CLI handlers are thin (validation + mapping + call application)  
- Run `make lint test` before pushing  
- Prefer small, focused PRs with clear commit messages  
- Update tests when changing domain behavior or repository contracts

## Questions?

Feel free to open a discussion or ask in the team chat.  
We can refine this document as the project evolves.

Happy coding — let's keep the architecture clean & the domain rich!

Last updated: March 2026