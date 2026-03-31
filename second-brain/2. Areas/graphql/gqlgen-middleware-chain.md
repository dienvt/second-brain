# gqlgen Middleware Chain & Resolver Invocation

> Explains how gqlgen invokes resolvers and chains middleware extensions,
> using `query { invoice(id: "x") { id segments { id name } } }` as the concrete example.

---

## Which Extension Implements What

| Extension         | `InterceptOperation` | `InterceptResponse` | `InterceptField` |
|-------------------|----------------------|---------------------|------------------|
| Log               | —                    | ✓                   | —                |
| Recovery          | —                    | —                   | ✓                |
| Database          | —                    | ✓                   | —                |
| Auth              | ✓                    | ✓ (no-op)           | —                |
| PolicyHydration   | ✓                    | —                   | —                |
| Authz             | ✓                    | —                   | —                |
| AccountDataloader | ✓                    | —                   | —                |

There are **three independent chains**, not one. Each extension only participates in the hooks it implements.

---

## Phase 1 — `InterceptOperation` (runs before field resolution)

gqlgen builds a nested call stack. The `next` func represents everything inner:

```
Auth.InterceptOperation(ctx)
│  1. shouldAllOperationsSkip? → NO (query.invoice is not in skippedOperations)
│  2. isLogoutMutation? → NO
│  3. getSessionCookie → valid cookie found
│  4. validateSessionCookie → active, no refresh needed
│  5. ParseBearerToken → token parsed
│  6. ctx = WithTokenContext(ctx, token)   ← token injected
│  calls next(ctx) ↓
│
PolicyHydration.InterceptOperation(ctx)
│  7. GetTokenFromContext(ctx) → token ✓
│  8. cacheGet(userID) → miss (first request)
│  9. AuthClient.FetchUserPolicies(ctx, userID) → [{effect:ALLOW, actions:[billing.report:manage], ...}]
│  10. cacheSet(userID, entry)
│  11. ctx = context.WithValue(ctx, ctxKeyPolicies{}, policies)  ← policies injected
│  calls next(ctx) ↓
│
Authz.InterceptOperation(ctx)
│  12. GetTokenFromContext(ctx) → token ✓
│  13. ExtractResolvers(operationCtx)
│       → only iterates top-level selection set
│       → returns ["query.invoice"]  ← NOT invoice.segments (not recursive)
│  14. "query.invoice" in skipAuthzOperations? → NO
│      "query.invoice" in skippedOperations? → NO
│  15. operationRegistry["query.invoice"]
│       → {RequiredAction: PermissionBillingReportManage, ExtractDRN: staticDRN("drn:*:billing.report:*")}
│  16. EvaluatePolicies(policies, "billing.report:manage", "drn:*:billing.report:*") → ALLOW
│  17. emitAuditLog(... ALLOW)
│  calls next(ctx) ↓
│
AccountDataloader.InterceptOperation(ctx)
│  18. ctx = dataloader.WithContext(ctx, DigimaClient)  ← dataloader injected
│  calls next(ctx) ↓
│
OperationHandler(ctx)                          ← gqlgen internal
   19. parses & validates the query AST
   20. withResponseContext(ctx, ...)           ← response context initialized HERE
   21. builds field execution plan (a ResponseHandler closure)
   returns ResponseHandler
```

At this point **no field resolver has run yet**. The `ResponseHandler` is just a plan.

---

## Phase 2 — `InterceptResponse` (wraps the ResponseHandler)

gqlgen takes the `ResponseHandler` from Phase 1 and wraps it with all `InterceptResponse` hooks:

```
Log.InterceptResponse(ctx)
│  22. generate requestId → ctx = WithFields(ctx, requestId)
│  23. log "Request process started"
│  24. defer: log "Request process finished" + elapsed time
│  calls next(ctx) ↓
│
Database.InterceptResponse(ctx)
│  25. ctx = ConfigureAccountDatabaseContext(ctx, 0)  ← DB connection in ctx
│  calls next(ctx) ↓
│
Auth.InterceptResponse(ctx)
│  26. no-op: return next(ctx)
│  calls next(ctx) ↓
│
ResponseHandler(ctx)    ← field resolution starts HERE
   ...
```

---

## Phase 3 — Field Resolution (inside the ResponseHandler)

Every field call — including scalars — goes through `ec._fieldMiddleware` in gqlgen's generated code,
which chains through `InterceptField`. Only **Recovery** implements this hook.

```
ResponseHandler executes field plan:

── field: query.invoice (has resolver) ──────────────────────────────
Recovery.InterceptField(ctx)
│  defer recover()   ← catches any panic inside
│  calls next(ctx) ↓
│    queryResolver.Invoice(ctx, "x")
│      → InvoiceController.ShowInvoice(ctx, "x")
│        → InvoiceService.Show(ctx, "x")    ← DB query via DatabaseManager
│          returns domainInvoice
│        → transformers.TransformInvoice(invoice)
│          returns *model.Invoice{ID:"x", Segments:[{Id:"s1"},{Id:"s2"}]}
returns *model.Invoice, nil

── field: invoice.id (scalar, auto-generated accessor) ──────────────
Recovery.InterceptField(ctx)
│  calls next(ctx) → generated code reads invoice.ID field
returns "x", nil

── field: invoice.segments (has resolver) ────────────────────────────
Recovery.InterceptField(ctx)
│  defer recover()
│  calls next(ctx) ↓
│    invoiceResolver.Segments(ctx, invoice)
│      → InvoiceController.ListInvoiceSegments(ctx, invoice)
│        → builds segmentIds = ["s1", "s2"] from invoice.Segments
│        → SegmentService.List(ctx, Filter{Ids:["s1","s2"]}, pagination)
│          returns []domainSegment
│        → segmentTransformer.TransformSegments(segments)
│          returns []*model.Segment{{ID:"s1",Name:"seg1"}, ...}
returns []*model.Segment, nil

── field: segment.id (scalar, for each segment) ─────────────────────
Recovery.InterceptField(ctx) → returns "s1", nil
Recovery.InterceptField(ctx) → returns "s2", nil

── field: segment.name (scalar, for each segment) ───────────────────
Recovery.InterceptField(ctx) → returns "seg1", nil
Recovery.InterceptField(ctx) → returns "seg2", nil

ResponseHandler returns:
*graphql.Response{Data: {"invoice": {"id":"x", "segments":[...]}}}
```

Log defer fires → `"Request process finished", time_elapsed: 42ms`

---

## Key Insight: `ExtractResolvers` is shallow

`resolvers.go` only walks the **top-level** `SelectionSet`, not recursively:

```go
for _, sel := range selection.SelectionSet {  // only top-level fields
    resolvers = append(resolvers, fmt.Sprintf("%s.%s", operation, field.Name))
}
```

So Authz only ever sees `["query.invoice"]`, never `["invoice.segments", "invoice.id", ...]`.
This is why nested resolvers don't need entries in `operationRegistry` — the permission check
is coarse-grained at the root operation level.

---

## Complete Flow Diagram

```
HTTP Request
     │
     ▼
[HTTP Middlewares]
  RateLimiting → CORS
     │
     ▼
─────────────────────────────────────────────────────────
PHASE 1: InterceptOperation  (validation & context setup)
─────────────────────────────────────────────────────────
  Auth            → validate session, inject token
  PolicyHydration → fetch policies from cache/IAM, inject policies
  Authz           → check top-level operation against policy
  AccountDataloader → init dataloader
  OperationHandler  → parse AST, init response context
     │
     │  returns ResponseHandler (execution plan)
     ▼
─────────────────────────────────────────────────────────
PHASE 2: InterceptResponse  (wrap the ResponseHandler)
─────────────────────────────────────────────────────────
  Log      → requestId, start log, defer finish log
  Database → configure DB context
  Auth     → no-op passthrough
     │
     ▼
─────────────────────────────────────────────────────────
PHASE 3: Field Resolution  (inside ResponseHandler)
─────────────────────────────────────────────────────────
  For each field with a resolver:
    Recovery.InterceptField
      └── resolver function (controller → service → transform)

  Scalars (no Go resolver):
    Recovery.InterceptField
      └── generated struct field access
     │
     ▼
*graphql.Response{Data: {...}}
     │
     ▼
HTTP Response
```

---

## Short-Circuit Behaviour (auth failure example)

When `Auth.InterceptOperation` detects an invalid session it returns a custom `ResponseHandler`
**without calling `next`**:

```
Auth.InterceptOperation → returns operationHandleError(...)
                            ↑
                       next() never called
                            ↓
  PolicyHydration, Authz, AccountDataloader → NEVER run
  OperationHandler                          → NEVER runs
  response context                          → NEVER initialized
```

Phase 2 (`InterceptResponse`) still runs and wraps the custom handler — Log and Database
hooks still execute. However `graphql.AddError(ctx, ...)` would panic inside the custom
handler because the response context was never initialized. The correct pattern is to build
`gqlerror.List` directly:

```go
return func(_ context.Context) *graphql.Response {
    if typed, ok := gqlErr.(*gqlerror.Error); ok {
        return &graphql.Response{Errors: gqlerror.List{typed}}
    }
    return &graphql.Response{Errors: gqlerror.List{{Message: gqlErr.Error()}}}
}
```
