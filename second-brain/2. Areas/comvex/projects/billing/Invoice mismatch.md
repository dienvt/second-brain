2. CLI: invoices:upsert
```
invoices:upsert all --from="2026-05-31T15:00:00Z" --to="2026-06-01T14:59:59.9999Z"
```
```
---
upsert segments

segments:create 499 all --from "2026-08-31T15:00:00Z" --to "2026-09-30T14:59:59.999999999Z" --recalculate

invoices:upsert 499 --from "2026-08-31T15:00:00Z" --to "2026-09-30T14:59:59.999999999Z"
---
invoices:create all --from="2026-05-31T15:00:00Z" --to="2026-06-01T14:59:59.9999Z"
```
  3. RabbitMQ job: V1RouteInvoicesUpsert

  Both of the above publish one job per account with this payload (pb/worker/invoices/v1/upsert_invoice.proto):

  message UpsertInvoice {
    uint64 account_id = 1;
    google.protobuf.Timestamp from = 2;
    google.protobuf.Timestamp to   = 3;
  }

  What the upsert does (domain/services/invoice/service.go:298)

  4. It looks up the existing invoice by account_id and the exact from/to. If none exists, it logs a warning and skips. It does not create one; use Create / invoices:create for that.
  5. It reloads the account's segments for that period and sorts them by display order.
  6. It recomputes the cost (sum of DiscountedCost) and the tax, then saves the invoice.