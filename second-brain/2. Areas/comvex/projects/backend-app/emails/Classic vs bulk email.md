# Classic vs bulk email

Both types live in the same `Email` model (`app/Models/Account/Email/Email.php`) and share the same send entry point (`ProcessEmailJob`) and the same envelope/dispatch primitives (`CreateEnvelopeJob`, `DispatchEnvelopeJob`, SendGrid). They diverge in how recipients are resolved and how envelopes are created and dispatched.

| | Classic (`TYPE_CLASSIC`) | Bulk (`TYPE_BULK`) |
|---|---|---|
| Recipients | Individual contacts / groups as to + cc + bcc | Contact groups (static + dynamic), to + excluded only |
| Envelopes | **One** envelope with all to/cc/bcc recipients | **One envelope per contact** (personalized) |
| Envelope creation | Synchronous in `ProcessEmailJob` (`createEnvelope()`) | Async, chunked: `PrepareEmailChunkJob` → N× `ProcessEmailChunkJob` |
| Dispatch | `ClassicEnvelopeCreatedEvent` → `DispatchClassicEnvelopeListener` → `dispatch_sync(DispatchEnvelopeJob)` — user gets immediate error feedback | `BulkEnvelopeCreatedEvent` → `DispatchBulkEnvelopeListener` (queued) → `dispatch(DispatchEnvelopeJob)` — fire-and-forget with retries |
| Failure handling | Full rollback: deletes envelope, recipients, events, statistics, links; email back to draft; throws `FailedToSendEmailException` | Envelope kept with `failed` recipient event (shows in stats), never sent; `DispatchFailedEvent` fired |
| Unsubscribe footer | Not included | Included by default (`OPTION_INCLUDE_FOOTER` — only applies when `isBulk()`) |
| Completion signal | Marked processed right after envelope creation | Redis `bulk_pending_contacts` counter reaches 0 across all chunk chains |
| Activity logging | Direct | Buffered in Redis via `BulkEmailSentActivityAccumulator`, bulk-flushed |
| Dispatched event | `ClassicEmailEnvelopeDispatchedEvent` | `BulkEmailEnvelopeDispatchedEvent` |
| Trace logging | `TraceIdContext` stage events (send_started, processing, preflight, envelope_created) | Standard log context only |

## Shared behavior

- **Scheduling:** both check `scheduled_at`; future sends are pushed to the SQS scheduler microservice, which calls back `/v2/emails/{id}/send-scheduled`.
- **Pre-flight validation** in `CreateEnvelopeJob`: free email domain sender, empty/malformed recipient address, exclusion list, missing placeholder values.
- **Domain validation** in `AbstractDispatchEnvelopeListener`: sender domain must be registered and valid, otherwise invalid-domain credits are decremented in production.
- **SendGrid responses** in `DispatchEnvelopeJob`: `202` marks recipients dispatched; `429` releases the job back to the queue with 5–10s jitter; anything else throws `SendgridException`.

## Related

- [[Bulk email flow]]
