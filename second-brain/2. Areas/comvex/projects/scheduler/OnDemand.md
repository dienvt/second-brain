---
tags: [comvex, scheduler, dynamodb, sqs, rabbitmq, lambda]
updated: 2026-10-07
---

# Scheduler — On-demand jobs

Lambda: `digima-staging-lambda-scheduler`
https://ap-northeast-1.console.aws.amazon.com/lambda/home?region=ap-northeast-1#/functions/digima-staging-lambda-scheduler?tab=code

Repo: `digima-backend-scheduler` — `app/app.go`, `app/events.go`, `domain/services/ondemand/service.go`, `infra/database/repositories/dynamodb/ondemand.go`

## TL;DR

Two completely separate triggers hit the **same** Lambda:

1. **SQS** → *store* the job in DynamoDB. Nothing is executed.
2. **EventBridge cron (one rule per `code`)** → *execute* every job of that code whose `scheduled_at` has passed.

> Analogy: a restaurant reservation book. Clients phone in reservations (SQS → DynamoDB). Nothing is cooked. A kitchen timer rings every minute (EventBridge); on each ring a cook opens the book to one dish type (`SMS_SEND`), pulls every reservation whose time has come, stamps it "cooking" so no other cook grabs it, makes it, stamps it "served".

## Flow

```
  WRITE PATH (any time)                         EXECUTE PATH (cron)
  ─────────────────────                         ───────────────────
  sms-api / email-api / line-api /              EventBridge rule, e.g. every 1 min
  agent / notification / mcp / backend-app      {"name":"aws_cloudwatch_event_rule_on_demand",
        │  SQS SendMessage                       "type":"SMS_SEND"}
        ▼                                             │
  ┌─────────────────┐                                 ▼
  │ scheduler Lambda│                          ┌─────────────────┐
  │ EventTypeSQS    │                          │ scheduler Lambda│
  │ ScheduleJob()   │                          │ EventTypeCloud- │
  └────────┬────────┘                          │ WatchOnDemand   │
           │ PutItem                            │ ProcessScheduled│
           ▼                                   │ Jobs("SMS_SEND")│
  ┌──────────────────────────────┐             └────────┬────────┘
  │ DynamoDB on-demand-schedules │   Query GSI          │
  │ Id (PK)                      │◄──CodeStatusIndex────┘
  │ CodeStatus="SMS_SEND#PENDING"│   CodeStatus = "SMS_SEND#PENDING"
  │ ScheduledAt (GSI sort key)   │   AND ScheduledAt <= now, Limit 1000
  │ Payload, HandlerOptions      │
  └──────────────────────────────┘
                 per job, in its own goroutine:
                 ① UpdateJob → PROCESSING   (condition: CodeStatus == "SMS_SEND#PENDING")
                 ② publish: http | rabbitmq (v6 envelope) | rabbitmq_v7 (headers)
                 ③ UpdateJob → PROCESSED / FAILED / back to PENDING
```

## Write path (SQS)

- Client does a bare `sqs.SendMessage` with body:
  ```json
  {"code":"SMS_SEND","scheduled_at":"…","unique_id":"…","payload":{…},
   "handler_options":{"driver":"rabbitmq","exchange":"sms.direct","routing_key":"v3.message.scheduled_send",
                      "bearer_type":"base64","account_id":1,"account_code":"…","user_id":2},
   "overwrite":false}
  ```
- `Event.UnmarshalJSON` (`events.go`) detects an SQS event → `handleSqsEvent` → `ScheduleJob` → `PutItem`.
- `unique_id` becomes the DynamoDB `Id`; without `overwrite` a duplicate `unique_id` hits `attribute_not_exists(Id)` and is logged as a **warning**, not an error (idempotent re-scheduling).
- Row stores `CodeStatus = "<CODE>#PENDING"` — a composite key so "due and pending for this code" is a single index range query.

## Execute path (EventBridge)

`domain/services/ondemand/service.go ProcessScheduledJobs(code)`:

1. Rule payload `{"name":"aws_cloudwatch_event_rule_on_demand","type":"sms_send","revertToPendingOnFailure":bool}` → `type` upper-cased → `code`. **One rule per code.** The scheduler never scans "all pending jobs".
2. Open **one** RabbitMQ publisher for the whole run. Take the Lambda deadline (or 12 min default) as a hard stop; stop paging when close to it.
3. Query GSI `CodeStatusIndex`: `CodeStatus = "<CODE>#PENDING" AND ScheduledAt <= now`, `Limit 1000`, cursor-paginated.
4. Fan out one goroutine per job (hence ADR-0001: Lambda memory 128 → 512 MB in prod).
5. Per job:
   - `UpdateJob → PROCESSING` with `ConditionExpression CodeStatus == "<CODE>#PENDING"`. **This conditional write is the lock.** If a slow run overlaps the next cron tick, the second one's condition fails → warn + skip → no double-send.
   - Dispatch on `handler_options.driver`:
     - `http` → HTTP call with OAuth/JWT bearer
     - `rabbitmq` → framework-**v6 envelope** in the body `{"data":…,"authorization":"Bearer <base64>","retry_count":0}`, no headers — for `sms-worker` / `email-worker` (still on v6)
     - `rabbitmq_v7` → raw payload body + AMQP headers `authorization`, `dgm-identity` — for v7/v8 workers
   - `UpdateJob → PROCESSED`, or `FAILED`; with `revertToPendingOnFailure` a failed publish goes back to `PENDING` so the next tick retries.
6. Next page until empty or deadline, then disconnect.

## Gotchas

- **Execution is polling, not timers.** A job fires on the first cron tick *after* `scheduled_at`, so latency is up to one cron interval.
- **No rule for a `code` ⇒ jobs sit in `PENDING` forever**, with no error anywhere — the write path accepted them happily. Check the EventBridge rules when a new code is introduced.
- `driver` is about the **consumer's** framework version (which wire format it can read), not the scheduler's. Producers on v8 (`sms-api`, `email-api`) still send `"rabbitmq"` because their workers are on v6. See step 5 above and the producer/consumer map on the Confluence page.
- Nothing logged identifies the calling service by name. `code` / `exchange` / `routing_key` identify it indirectly; SQS `attributes.SenderId` (IAM role id, stamped by AWS) is in the `Received SQS Event` log dump. No producer sets `MessageAttributes`.
- `ScheduleJob` logs `args.HandlerOptions.Driver` as a proper nested field (queryable in CloudWatch Insights); the `Received SQS Event` line has the driver inside the escaped `body` string (not queryable).

## Related

- [[digima-backend-scheduler]] — `Event` struct, how `UnmarshalJSON` picks the event type, Lambda invocation lifecycle
- [[README]] — mermaid overview; the **fixed** path (`aws_cloudwatch_event_rule_fixed`) carries `payload` + `handler_options` in the rule itself and never touches DynamoDB
- Confluence: [Scheduler — Upgrade go-framework to v8](https://comvex.atlassian.net/wiki/x/KYAA0g), [Upgrade go-framework to v8 (guide)](https://comvex.atlassian.net/wiki/x/IAC4v)
- `docs/adr/0001-add-dynamodb-query-limit-and-lambda-memory-scaling.md`
