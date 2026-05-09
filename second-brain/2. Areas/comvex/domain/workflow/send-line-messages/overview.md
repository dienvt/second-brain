# Workflow: Send LINE Message

## Overview

When a user sets up a workflow with a **Send LINE Message** step, the system requires a `GroupMessage` to exist in the LINE microservice before the workflow action can be saved. The `GroupMessage` is the template that defines what content will be sent; the workflow action only holds a reference to it via `resource_id`.

The `GroupMessage` lives entirely in **`digima-backend-line-api`** — there is no local Eloquent model for it in `digima-backend-app`. All access goes through gRPC.

---

## Setup Flow (User Perspective)

```
1. User creates a GroupMessage  →  POST /api/v3/line/group-messages
                                      (creation_type: GROUP_MESSAGE_CREATION_TYPE_WORKFLOW)
                                      returns GroupMessage ID

2. User creates workflow action  →  POST /api/v2/workflow-actions
                                      type: send_line
                                      resource_id: <GroupMessage ID from step 1>
```

The workflow action **will be rejected** if the referenced GroupMessage is not in `GROUP_MESSAGE_STATUS_PROCESSED` status at creation time.

---

## GroupMessage Model

Stored in: `group_messages` table (MySQL, in `digima-backend-line-api`)
Protobuf: `line/api/v1/group_messages.proto`
PHP class: `DigimaBackendLineApiProto\V1\GroupMessage`

| Field | Type | Notes |
|---|---|---|
| `id` | uint64 | Primary key in LINE service |
| `author_id` | uint64 | Digima user who created it |
| `sender_digima_user_id` | uint64? | User who sent it — always `nil` for workflow type |
| `status` | `GroupMessageStatus` | See statuses below |
| `creation_type` | `GroupMessageCreationType` | `WORKFLOW` or `BULK` |
| `messages` | `Message[]` | Up to 3 message items |
| `recipients` | `GroupMessageRecipients`? | Used for BULK type only; `nil` for workflow |
| `workflow_id` | uint64? | The workflow this belongs to (non-null = workflow type) |
| `channel_id` | uint64? | LINE channel |
| `scheduled_at` | Timestamp? | For scheduled sends (BULK only) |
| `sent_at` | Timestamp? | Updated to the latest recipient event's `occurred_at` |
| `statistics` | `GroupMessageStatistics`? | `sent_count`, `clicked_count` |

### GroupMessageStatus

| Value | Meaning |
|---|---|
| `GROUP_MESSAGE_STATUS_DRAFT` | Being edited, not ready |
| `GROUP_MESSAGE_STATUS_PROCESSING` | Being processed (BULK only) |
| `GROUP_MESSAGE_STATUS_PROCESSED` | Ready — **auto-set on creation for workflow type; required before linking to a workflow action** |
| `GROUP_MESSAGE_STATUS_SCHEDULED` | Scheduled for future send (BULK only) |

### GroupMessageCreationType

| Value | Meaning |
|---|---|
| `GROUP_MESSAGE_CREATION_TYPE_WORKFLOW` | Template for a workflow action; fired per-contact one-at-a-time |
| `GROUP_MESSAGE_CREATION_TYPE_BULK` | One-time broadcast to many contacts at once |

### Key behavioural differences: Workflow vs Bulk

| | Bulk | Workflow |
|---|---|---|
| Trigger | Human manually sends/schedules | Workflow engine fires per-contact |
| Recipients | All at once (`ContactIds` / groups / all friends) | One contact per invocation |
| Status on create | `draft` / `processing` / `scheduled` | Always auto-set to `processed` |
| `SenderDigimaUserId` | Set to the activating user | Always `nil` |
| `SentAt` | Set at send time | Updated to latest recipient event `occurred_at` |

---

## Message (content inside GroupMessage)

Each GroupMessage contains one or more `Message` items (max 3).

### MessageType

| Value | Content fields |
|---|---|
| `MESSAGE_TYPE_TEXT` | `body` (string) — supports `*\|user.display_name\|*` placeholder |
| `MESSAGE_TYPE_IMAGE` | `shared_file_id`, `image_url`, `shared_file_name` |
| `MESSAGE_TYPE_RICH` | `shared_file_id`, `image_url`, `link_url` |

---

## Workflow Action (backend-app side)

Stored in `workflow_actions` table as an Eloquent `Action` model.

| Column | Value for send_line |
|---|---|
| `type` | `send_line` |
| `resource_id` | GroupMessage ID (in LINE microservice) |
| `resource_type` | `null` (no local morph map for LINE) |

Validation rule at action creation (`CreateActionRequest`):
```
resource_id: exists_in_line_account:group_messages,id,status,GROUP_MESSAGE_STATUS_PROCESSED
```

---

## Execution Flow

### 1. digima-backend-app side

When a workflow fires the `send_line` action for a participant:

```
ExecuteActionJob::handle()
  └─ ActionResolver::resolveExecutor(action, participant)
       └─ SendLineActionExecutor::execute()
            ├─ groupMessageId = action->resource_id
            ├─ contactId = participant->contact_id
            └─ Workflow(account)->send(groupMessageId, contactId)
                 └─ gRPC: WorkflowsServiceClient::Send
                          { group_message_id, contact_id }
                    → line-api handles the rest

  └─ action->participants()->updateExistingPivot(
         participant->id, ['processed_at' => now()]
     )
```

Guard conditions (action is skipped if any are true):
- `participant.unenrolled_at` is set
- Workflow status is not `STATUS_ENABLED`
- Action-participant pivot `processed_at` already set (deduplication via cache lock)
- `shouldReschedule()` returns true (outside workflow working hours)

Feature flag required: `FEATURE_FLAG_LINE_WORKFLOW` must be enabled on the account.

### 2. digima-backend-line-api side

On receiving `WorkflowsService.Send(group_message_id, contact_id)`:

```
WorkflowsService.Send(groupMessageId, contactId)
  │
  ├─ Lookup LINE user for contactId (must be a follower)
  │
  ├─ [No LINE user found]
  │    └─ Create GroupMessageRecipient (user_id = nil)
  │         └─ Create GroupMessageRecipientEvent(type=failed, reason="not_line_user")
  │
  └─ [LINE user exists]
       └─ Create GroupMessageRecipient (user_id = user.id)
            └─ WorkerClient.RecipientContactMessageCreate(recipient_id, group_message_id)
                 │
                 ▼ (worker picks up the job)
                 ContactMessageService.CreateFromGroupMessage(group_message_id, recipient_id)
                   └─ Creates contact_messages rows (1 per message × 1 recipient)
                        └─ ContactMessageService.SendByRecipient(recipient_id)
                             └─ WorkerClient.ContactMessagesSend → LINE API delivery
```

---

## Data Model Chain

This chain lets you trace any `contact_message` back to the originating workflow:

```
contact_messages.group_message_recipient_id
        ↓
group_message_recipients.group_message_id
        ↓
group_messages.workflow_id  ←  non-null = triggered by a workflow
```

### Determining message origin

| `group_message_recipient_id` | `group_messages.workflow_id` | Meaning |
|---|---|---|
| `NULL` | — | Direct 1-on-1 send or inbound message |
| non-null | `NULL` | Bulk group message send |
| non-null | non-null | **Workflow-triggered send** |

### SQL to trace a contact_message back to its workflow

```sql
SELECT
    cm.id            AS contact_message_id,
    gmr.id           AS recipient_id,
    gmr.contact_id,
    gm.id            AS group_message_id,
    gm.workflow_id
FROM contact_messages cm
JOIN group_message_recipients gmr ON gmr.id = cm.group_message_recipient_id
JOIN group_messages gm            ON gm.id  = gmr.group_message_id
WHERE gm.workflow_id IS NOT NULL
  AND cm.id = <your_id>;
```

---

## Clone Workflow

When a workflow is cloned (`CloneWorkflowJob`), the `send_line` action's GroupMessage is also cloned in the LINE microservice:

```
cloneLine(groupMessageId):
  1. GroupMessages::show(groupMessageId)          ← fetch original
  2. Strip: id, author, recipients, dates, stats
  3. Set workflow_id = clonedWorkflow->id
  4. GroupMessages::create(data)                  ← create new GroupMessage (auto status=processed)
  5. Return new GroupMessage ID → stored as resource_id on cloned action
```

On clone failure, the cloned GroupMessage is deleted as part of cleanup.

---

## API Endpoints

### GroupMessage CRUD (LINE microservice proxy)

| Method | Endpoint | Handler |
|---|---|---|
| `GET` | `/api/v3/line/group-messages` | `GroupMessagesController@index` |
| `POST` | `/api/v3/line/group-messages` | `GroupMessagesController@store` |
| `GET` | `/api/v3/line/group-messages/{id}` | `GroupMessagesController@show` |
| `PUT` | `/api/v3/line/group-messages/{id}` | `GroupMessagesController@update` |
| `DELETE` | `/api/v3/line/group-messages/{id}` | `GroupMessagesController@destroy` |

### Workflow Action CRUD

| Method | Endpoint | Handler |
|---|---|---|
| `POST` | `/api/v2/workflow-actions` | `Workflow\ActionsController@store` |
| `PUT` | `/api/v2/workflow-actions/{id}` | `Workflow\ActionsController@update` |
| `DELETE` | `/api/v2/workflow-actions/{id}` | `Workflow\ActionsController@destroy` |

### Internal (EventBridge callback)

| Method | Endpoint | Handler |
|---|---|---|
| `POST` | `/internal/v2/workflow-actions/{id}/execute` | `WorkflowActionsController@execute` |

### gRPC (digima-backend-line-api)

| Service | Method | Called by |
|---|---|---|
| `GroupMessagesService` | `Create` | backend-app when user sets up workflow action |
| `WorkflowsService` | `Send(group_message_id, contact_id)` | backend-app `SendLineActionExecutor` at execution time |

---

## Key Files

### digima-backend-app

| File | Purpose |
|---|---|
| `app/Services/LineClient/Handlers/GroupMessages.php` | gRPC client for GroupMessage CRUD |
| `app/Services/LineClient/Handlers/Workflow.php` | gRPC client for workflow send |
| `app/Services/Workflow/Action/SendLineActionExecutor.php` | Executes the send_line action |
| `app/Services/Workflow/Action/ActionResolver.php` | Routes action type to executor |
| `app/Jobs/Workflow/Action/ExecuteActionJob.php` | Main job that runs a workflow action |
| `app/Jobs/Workflow/CloneWorkflowJob.php` | Handles cloning incl. GroupMessage clone |
| `app/Http/Requests/User/Line/CreateGroupMessageRequest.php` | Validation for GroupMessage creation |
| `app/Http/Requests/User/Workflow/Action/CreateActionRequest.php` | Validation for action creation (incl. send_line rule) |
| `app/Transformers/Account/Line/GroupMessageTransformer.php` | API response shape for GroupMessage |

### digima-backend-line-api

| File | Purpose |
|---|---|
| `domain/models/group_message.go` | GroupMessage domain model + IsBulkType / IsWorkflowType helpers |
| `domain/services/group_message/service.go` | GroupMessage CRUD; status transitions; enriches with Digima user info |
| `domain/services/workflow/service.go` | Handles `WorkflowsService.Send`; creates recipient; dispatches worker job |
| `domain/services/contact_message/service.go` | `CreateFromGroupMessage` (creates contact_message rows); `SendByRecipient` (delivers via LINE) |
| `interface/controllers/v1/workflow/controller.go` | gRPC controller for `WorkflowsService` |
| `infra/database/repositories/mysql/group_message/entity.go` | `group_messages` table mapping |
| `infra/database/repositories/mysql/group_message_recipient/entity.go` | `group_message_recipients` table mapping |
| `infra/database/repositories/mysql/contact_message/entity.go` | `contact_messages` table mapping (`group_message_recipient_id` FK) |
