---
tags: [comvex, digima, backend, domain, workflow, automation]
project: digima-backend-app
source: app/Models/Account/Workflow/Workflow.php
created: 2026-04-23
---

# Workflow

Digima's **Workflow** feature is a per-tenant automation engine: enrol contacts based on filters, then execute a chain of actions (send email / SMS / LINE, delay, await event, branch on condition, send in-app notification). Each [[Account]] can have many workflows, gated by `FEATURE_WORKFLOW` subscription.

## TL;DR — which service drives it?

**The monolith itself**, not a dedicated workflow microservice. There is **no `WorkflowClient` and no `workflow-api`** — confirmed by grep across `app/` and `app/Providers/GrpcClientServiceProvider.php`.

Execution is split across three layers:

| Layer | Where | Role |
|---|---|---|
| **Orchestration** | Laravel queue + jobs + listeners in `digima-backend-app` itself | Enrolment, trigger matching, action chaining, next-action dispatch, statistics |
| **Scheduling of delayed/scheduled actions** | **AWS SQS (`scheduler-on-demand`) → AWS EventBridge Scheduler** | At `scheduled_at`, EventBridge POSTs to the monolith's internal API `/internal/v2/workflow-actions/{id}/execute` |
| **Action side-effects** | Microservices called from executors: `email-api`, `sms-api`, `line-api` | Actual delivery of email / SMS / LINE messages |

So "which service drives workflow" answered three ways:
- **Orchestration** → this monolith.
- **Timing / scheduling** → AWS EventBridge + SQS (generic AWS infra, not a Digima service).
- **Message delivery** → the respective `*-api` microservices, via each action executor.

---

## 1. Data model

Tables live in the **per-account MySQL DB** (`dgm_account_{id}`). Statistics live in the per-account MongoDB.

```
Workflow ─┬─< Trigger     >─ ContactFilter
          ├─< Action ──────── self-join via previous_action_id (chain)
          │                   └─ morph resource → Email / Notification / SmsMessage / LineGroupMessage / ContactFilter
          └─< Participant (= Contact enrolled in Workflow)
                  └─< ActionParticipant pivot ─ scheduled_at / processed_at
```

### Models

| Model | File | Table | Key fields |
|---|---|---|---|
| `Workflow` | `app/Models/Account/Workflow/Workflow.php` | `workflows` | `id`, `name`, `status` (`draft` / `enabled` / `disabled` / `invalid` / `creation_in_progress`), `user_id`, `settings` JSON (`days_of_week`, `time_of_day`, `re-enroll`), soft-deletes |
| `Action` | `app/Models/Account/Workflow/Action.php` | `workflow_actions` | `id`, `type`, `workflow_id`, `previous_action_id`, `previous_action_case`, morph `resource_id/resource_type`, `parameters` JSON |
| `Trigger` | `app/Models/Account/Workflow/Trigger.php` | `workflow_triggers` | `id`, `type` (`enrollment` / `unenrollment`), `workflow_id`, `contact_filter_id` |
| `Participant` | `app/Models/Account/Workflow/Participant.php` | `workflow_participants` | `id`, `contact_id`, `workflow_id`, `unenrolled_at`, `completed_at`, `re_enrollment_hash`, `uid` |
| `ActionParticipant` | `app/Models/Account/Workflow/ActionParticipant.php` | `workflow_action_participant_pivot` | `workflow_action_id`, `workflow_participant_id`, `scheduled_at`, `processed_at` |
| `Notification` | `app/Models/Account/Workflow/Notification.php` | `workflow_notifications` | `id`, `subject`, `body_html`, `body_text`, `set_person_in_charge_as_recipient` |
| `Statistic` | `app/Models/Account/Workflow/Statistic.php` | `workflow_statistics` (MongoDB) | `participant_id`, `workflow_id`, `contact_id`, `event_name`, `created_at` |

### Action types — `Action.php` L48-54

| Constant            | String         | Purpose                                                                       |
| ------------------- | -------------- | ----------------------------------------------------------------------------- |
| `TYPE_DELAY`        | `delay`        | Wait N time units                                                             |
| `TYPE_SEND_EMAIL`   | `send_email`   | Send email via `EnvelopeCreator` (resource = Email)                           |
| `TYPE_SEND_SMS`     | `send_sms`     | Send SMS via `sms-api` (resource_id = message_id)                             |
| `TYPE_SEND_LINE`    | `send_line`    | Send LINE via `line-api` (resource_id = group_message_id)                     |
| `TYPE_NOTIFICATION` | `notification` | In-app user notification (resource = WorkflowNotification)                    |
| `TYPE_CONDITION`    | `condition`    | Branch: evaluate contact filter, pick `previous_action_case` `true` / `false` |
| `TYPE_AWAIT_EVENT`  | `await_event`  | Pause until an event fires                                                    |

### Await-event sub-types — `Action.php` L38-43

`email_link_clicked`, `form_submitted`, `portal_submitted`, `sms_link_clicked`, `sms_received`, `web_tracking_visited`.

### Trigger types — `Trigger.php` L31-32

`enrollment`, `unenrollment`.

---

## 2. Execution architecture

### Two ways a workflow starts executing

**A. Daily scheduled sweep** (Laravel scheduler)

```
Kernel.php  ──┬─▶ ExecuteTriggersScheduler (app/Schedulers/Workflow/ExecuteTriggersScheduler.php L9-34)
              │     ↳ ->dailyAt('03:00')->timezone('Asia/Tokyo')->onOneServer()
              │
              └─▶ artisan digima:workflow-triggers-dispatch all
                  ↳ ExecuteWorkflowTriggersDispatcherCommand (app/Console/Commands/Workflow/)
                    ↳ per enabled account with FEATURE_WORKFLOW:
                        dispatch_sync(new ExecuteAllTriggerJob)
                          ↳ per Trigger on each enabled Workflow:
                              enrollment  → PrepareEnrollmentJob   → EnrollContactsToWorkflowJob
                              unenrollment→ PrepareUnenrollmentJob → UnenrollContactsFromWorkflowJob
```

**B. Event-driven** (real time)

```
Contact-related events ─▶ app/Listeners/Workflow/MatchSubscriber.php
                              ↳ fires Workflow\ContactMatchedEvent
                                    ↳ ExecuteTriggerSubscriber::whenMatched (L26)
                                        ↳ dispatch ExecuteTriggerJob
```

`MatchSubscriber` listens to outgoing/incoming calls, form submissions, portal submissions, web-tracking visits, contact status changes, etc.

### Once a participant is enrolled — action chain

```
EnrollContactsToWorkflowJob (app/Jobs/Workflow/EnrollContactsToWorkflowJob.php L14-100)
  │ insertOrIgnore participants
  │ fire Workflow\Participant\EnrolledEvent  (L59)
  └─▶ DispatchNextActionJob (app/Jobs/Workflow/Action/DispatchNextActionJob.php L17-150)
        │ find next action via previous_action_id chain (L93-96)
        │ if no next → participant.completed_at = now + fire CompletedEvent (L100-104)
        │ else check unenrollment filter; if match → complete (L120-148)
        └─▶ CreateActionParticipantPivotJob (app/Jobs/Workflow/Action/CreateActionParticipantPivotJob.php L11-172)
              │ validates feature subscription per action type (EMAIL / SMS / LINE)
              ├─ if Action.type = DELAY:
              │     attach pivot with scheduled_at = now + delay
              │     dispatch_sync(PublishActionParticipantJob) ─▶ AWS EventBridge path (see below)
              └─ else:
                    attach pivot and dispatch ExecuteActionJob immediately
                          │
                          ▼
ExecuteActionJob (app/Jobs/Workflow/Action/ExecuteActionJob.php L15-149)
  │ ShouldQueue, up to 15 retries w/ incremental backoff (L27, 58-61)
  │ Cache lock per (action_id, participant_id) to dedupe (L72-77)
  │ shouldReschedule()? ─ outside business hours → reschedule to next slot (L114-131)
  │ ActionResolver::resolveExecutor(action, participant)
  │ $executor->execute()
  │ fire Workflow\Action\ExecutedEvent (L147)
  └─▶ (listener) DispatchNextActionJob → loop
```

### Delay / scheduling — the AWS EventBridge path

This is the **only time Digima's workflow leaves the monolith**.

```
CreateActionParticipantPivotJob (type=DELAY branch, L47-64)
   ↳ scheduledAt = now + delay  (ceilMinute)
   ↳ attach pivot scheduled_at
   ↳ dispatch_sync(PublishActionParticipantJob)
         │
         ▼
PublishActionParticipantJob (app/Jobs/Workflow/Action/PublishActionParticipantJob.php L44-79)
   │ builds SQS message:
   │   code         = "workflow_action_execute" (optionally "_p1".."_pN" partitioned by participant_id)
   │   scheduled_at = RFC3339
   │   payload      = {action_id, participant_id, scheduled_at}
   │   handler_options = {driver: http, method: POST,
   │                      url:  {internal_api}/v2/workflow-actions/{actionId}/execute,
   │                      headers: [Authorization, DGM-ACCOUNT-ID]}
   └─▶ Queue::connection('scheduler-on-demand')->pushRaw(...)
          │
          ▼
AWS SQS queue `{prefix}scheduler-on-demand`  (config/queue.php L123-128, driver: sqs)
          │
          ▼
AWS EventBridge Scheduler Rule(s)  (see L95 comment: "distribute ... to multiple EventBridge Rule")
          │   - at scheduled_at
          ▼
HTTP POST /internal/v2/workflow-actions/{id}/execute
          │
          ▼
WorkflowActionsController@execute (app/Http/Controllers/Internal/v2/WorkflowActionsController.php L15-80)
   │ FEATURE_WORKFLOW check (L31)
   │ look up pivot (action_id, participant_id) where processed_at IS NULL (L58-64)
   └─▶ dispatch ExecuteActionJob (L74)  ─ back into the normal action pipeline
```

The partitioning (`_p1` .. `_pN`) is controlled by `config('workflow.scheduler_partition_count')`; round-robin by `participant_id % (N+1)` to spread load across multiple EventBridge Rules (`PublishActionParticipantJob.php` L84-109).

### Safety-net for missed scheduled actions

`app/Console/Commands/Workflow/DispatchWorkflowScheduledActionsCommand.php` (L16-152) — `digima:workflow-scheduled-actions-dispatch {account} {from} {to}`. Scans `workflow_action_participant_pivot` for rows with `scheduled_at` in range and `processed_at IS NULL`, re-dispatches `ExecuteActionJob`.

---

## 3. Action executors

Under `app/Services/Workflow/Action/`.

| Executor | Outbound dependency | Notes |
|---|---|---|
| `EmailActionExecutor` | **Local** — `EnvelopeCreator` creates email envelope in this monolith's DB | `email-api` later picks up sending via its own flow (not called from executor) |
| `SendSmsActionExecutor` | **External** — `SmsClient\Handlers\Workflow->send([message_id, contact_id])` → POST `sms-api/workflow/send` | |
| `SendLineActionExecutor` | **External** — `LineClient\Handlers\Workflow->send(groupMessageId, contactId)` → `line-api` | Registered in `GrpcClientServiceProvider.php` L198-210 |
| `NotificationActionExecutor` | Local — creates activity + in-app notifications for assigned users | |
| `ConditionActionExecutor` | Local — `Filterer` evaluates contact filter; sets `previous_action_case` for branch | |
| `DelayActionExecutor` | n/a — marks pivot `processed_at`; scheduling already happened in `CreateActionParticipantPivotJob` | |
| `AwaitEventExecutor` | Local — holds participant until awaited event fires (processed_at set when event matches) | |

Dispatch: `ActionResolver::resolveExecutor($action, $participant)` in `app/Services/Workflow/Action/ActionResolver.php` L9-40.

Base class `AbstractActionExecutor::shouldReschedule(Carbon $dt)` (L91-111) enforces workflow `days_of_week` + `time_of_day` windows — if outside, the action is rescheduled.

---

## 4. Events

### Produced (in-process Laravel events, no Kafka)

| Event | Fired at | Listeners |
|---|---|---|
| `Workflow\EnabledEvent` | on enable | `EnrollContactsToWorkflowSubscriber::whenMatched` → PrepareEnrollmentJob |
| `Workflow\ContactMatchedEvent` | from `MatchSubscriber` on contact events (call, form, portal, web, status) | `ExecuteTriggerSubscriber::whenMatched` |
| `Workflow\Trigger\UpdatedEvent` | trigger filter changed | `EnrollContactsToWorkflowSubscriber` re-enrolls |
| `Workflow\Action\ExecutedEvent` | action completed | `UpdateStatisticSubscriber`, chains to next action |
| `Workflow\Participant\EnrolledEvent` | participant created | stats |
| `Workflow\Participant\UnenrolledEvent` | participant unenrolled | stats |
| `Workflow\Participant\CompletedEvent` | no next action | `ExecuteTriggerSubscriber::whenCompleted` — may re-enrol if `settings['re-enroll']` |

**No Kafka topics.** All workflow events stay in-process.

### Consumed

`MatchSubscriber` listens to contact-side events from across the codebase (calls, forms, emails opened/clicked, status changes, etc.) and converts them into `Workflow\ContactMatchedEvent`.

---

## 5. Feature gating

- Subscription code: **`FEATURE_WORKFLOW`** — `app/Models/Account/Feature/Subscription.php`.
- Checked at every entry point:
  - `WorkflowActionsController.php:31` — rejects with `workflow_feature_not_subscribed` error.
  - `ExecuteWorkflowTriggersDispatcherCommand.php:72`.
  - `ExecuteTriggerSubscriber.php:37`.
  - All `WorkflowPolicy` / `ActionPolicy` / `TriggerPolicy` checks.
- On `Feature\Subscription\EndedEvent` for `workflow` → `DisableWorkflowsListener` disables all workflows.
- On subscription re-start → `ResumeWorkflowActionsSubscriber` resumes paused pivots.
- Per-account cap: `Account.max_workflow_count` enforced by `MaxWorkflowCountExceededException`.

---

## 6. HTTP endpoints

User API — `app/Http/Controllers/User/v2/Workflow/WorkflowsController.php` (L18-128):

| Method | Path | Job dispatched |
|---|---|---|
| GET | `/workflows` | list (filters, pagination) |
| GET | `/workflows/{id}` | show |
| POST | `/workflows` | `CreateWorkflowJob` |
| PATCH | `/workflows/{id}` | `UpdateWorkflowJob` |
| DELETE | `/workflows/{id}` | `DeleteWorkflowJob` |
| POST | `/workflows/{id}/clone` | `CloneWorkflowJob` |

There are sibling controllers for actions / triggers / notifications under the same namespace.

Internal API (EventBridge callback) — `app/Http/Controllers/Internal/v2/WorkflowActionsController.php`:
- `POST /internal/v2/workflow-actions/{id}/execute` — dispatches `ExecuteActionJob`.

---

## 7. Key files

### Models — `app/Models/Account/Workflow/`
`Workflow.php`, `Action.php`, `Trigger.php`, `Participant.php`, `ActionParticipant.php`, `Notification.php`, `Statistic.php`.

### Services — `app/Services/Workflow/Action/`
`AbstractActionExecutor.php`, `ActionResolver.php`, `ActionScheduler.php`, `EmailActionExecutor.php`, `SendSmsActionExecutor.php`, `SendLineActionExecutor.php`, `NotificationActionExecutor.php`, `ConditionActionExecutor.php`, `DelayActionExecutor.php`, `AwaitEventExecutor.php`, `Notification/PlaceholderHelper.php`.

### Jobs — `app/Jobs/Workflow/`

Top level: `CreateWorkflowJob`, `UpdateWorkflowJob`, `DeleteWorkflowJob`, `CloneWorkflowJob`, `EnrollContactsToWorkflowJob`, `UnenrollContactsFromWorkflowJob`, `PrepareEnrollmentJob`, `PrepareUnenrollmentJob`.

`Action/`: `ExecuteActionJob`, `DispatchNextActionJob`, `CreateActionParticipantPivotJob`, `PublishActionParticipantJob`, `CreateActionJob`, `UpdateActionJob`, `DeleteActionJob`, `ResumeActionJob`.

`Trigger/`: `ExecuteAllTriggerJob`, `ExecuteTriggerJob`, `CreateTriggerJob`, `UpdateTriggerJob`, `DeleteTriggerJob`.

### Listeners — `app/Listeners/Workflow/`
`EnrollContactsToWorkflowSubscriber.php`, `ExecuteTriggerSubscriber.php`, `UpdateStatisticSubscriber.php`, `MatchSubscriber.php`, `DisableWorkflowsListener.php`, `ResumeWorkflowActionsSubscriber.php`, `UnenrollArchivedContactsListener.php`, `UnenrollDuplicatedContactsListener.php`, `DispatchNextActionListener.php`.

### Events — `app/Events/Workflow/`
`EnabledEvent.php`, `ContactMatchedEvent.php`, `Action/ExecutedEvent.php`, `Action/CascadedEvent.php`, `Participant/{Enrolled,Unenrolled,Completed}Event.php`, `Trigger/UpdatedEvent.php`.

### Scheduler — `app/Schedulers/Workflow/ExecuteTriggersScheduler.php`

### Commands — `app/Console/Commands/Workflow/`
`ExecuteWorkflowTriggersDispatcherCommand.php`, `DispatchWorkflowScheduledActionsCommand.php`, `UpdateWorkflowActionEmailPlaceholderValidationCommand.php`, `ClearParticipantFirstReenrollmentHash.php`, `UpdateActiveParticipantCreatedAt.php`, `UnenrollDuplicatedActiveParticipantsCommand.php`.

### Migrations — `database/migrations/account/mysql/`
`2020_04_27_…_create_workflows_table`, `…_workflow_triggers`, `…_workflow_actions`, `…_workflow_participants`, `…_workflow_action_participant_pivot`, `…_workflow_notifications`, `…_workflow_notification_user_pivot`, plus several `alter_*` migrations through 2023.

### Config
- `config/queue.php` L123-128 — the `scheduler-on-demand` SQS connection.
- `config/workflow.php` — `scheduler_partition_count` for EventBridge Rule fan-out.

---

## Related

- [[Account]] — workflow data lives per-tenant; subscription gates access.
- [[FranchiseAccount]] — franchise contact sync and workflows are independent.
