# Workflow Action Execute Trigger Message Flow

  

```mermaid

graph TD

%% Workflow Action Scheduling

A[Workflow Action Scheduled] --> B[ExecuteActionJob]

B --> C{Should Reschedule?}

C -->|Yes| D[PublishActionParticipantJob]

C -->|No| E[Execute Action Directly]

  

%% Message Production

D --> F[Create Message Payload]

F --> G[Push to scheduler-on-demand Queue]

  

%% Message Structure

G --> H[Message Content:<br/>code: 'workflow_action_execute'<br/>unique_id: md5 hash<br/>scheduled_at: RFC3339<br/>payload: action_id, participant_id, scheduled_at<br/>handler_options: HTTP config]

  

%% External Lambda Consumer

H --> I[AWS Lambda Scheduler Microservice]

I --> J[Monitor scheduler-on-demand Queue]

J --> K[Process workflow_action_execute Messages]

  

%% HTTP Request to Internal API

K --> L[HTTP POST Request]

L --> M[Internal API Endpoint:<br/>/v2/workflow-actions/action_id/execute]

  

%% Internal API Processing

M --> N[WorkflowActionsController.execute]

N --> O{Feature Flag Check:<br/>WORKFLOW_ACTION_EXECUTE_SCHEDULER_LAMBDA}

O -->|Enabled| P[Validate Request Parameters]

O -->|Disabled| Q[Return Error: requires_feature_flag]

  

%% Database Validation

P --> R[Query workflow_action_participant_pivot Table]

R --> S{Pivot Data Exists?}

S -->|Yes| T[Create Participant Object]

S -->|No| U[Return Error: not_found]

  

%% Action Execution

T --> V[Dispatch ExecuteActionJob]

V --> W[ExecuteActionJob.handle]

W --> X[Cache Lock Check]

X --> Y[Process Action]

Y --> Z[Fire ExecutedEvent]

  

%% Action Types

Y --> AA{Action Type}

AA -->|SEND_EMAIL| BB[EmailActionExecutor]

AA -->|SEND_SMS| CC[SendSmsActionExecutor]

AA -->|SEND_LINE| DD[SendLineActionExecutor]

AA -->|NOTIFICATION| EE[NotificationActionExecutor]

AA -->|DELAY| FF[DelayActionExecutor]

AA -->|CONDITION| GG[ConditionActionExecutor]

AA -->|AWAIT_EVENT| HH[AwaitEventExecutor]

  

%% Event Handling

Z --> II[DispatchNextActionListener]

II --> JJ[DispatchNextActionJob]

JJ --> KK[Process Next Action in Workflow]

  

%% Styling

classDef queue fill:#e1f5fe

classDef lambda fill:#f3e5f5

classDef api fill:#e8f5e8

classDef job fill:#fff3e0

classDef executor fill:#fce4ec

  

class G,H queue

class I,J,K lambda

class M,N,P,R,S,T,U api

class B,V,W,X,Y,Z job

class BB,CC,DD,EE,FF,GG,HH executor

```

  

## Detailed Flow Description

  

### 1. **Workflow Action Scheduling**

  

- When a workflow action needs to be scheduled, `ExecuteActionJob` checks if it should be rescheduled

- If rescheduling is needed, `PublishActionParticipantJob` is dispatched

  

### 2. **Message Production**

  

- `PublishActionParticipantJob` creates a message with:

- `code: 'workflow_action_execute'`

- `unique_id: md5 hash of the message`

- `scheduled_at: RFC3339 timestamp`

- `payload: action_id, participant_id, scheduled_at`

- `handler_options: HTTP configuration for internal API call`

  

### 3. **Queue Publishing**

  

- Message is pushed to `scheduler-on-demand` queue using AWS SQS

- Queue message ID is logged for tracking

  

### 4. **External Lambda Consumer**

  

- AWS Lambda scheduler microservice monitors the `scheduler-on-demand` queue

- Processes messages with `code: 'workflow_action_execute'`

- Extracts handler options and makes HTTP request

  

### 5. **Internal API Processing**

  

- Lambda makes HTTP POST request to `/v2/workflow-actions/{action_id}/execute`

- `WorkflowActionsController.execute()` handles the request

- Validates feature flag `WORKFLOW_ACTION_EXECUTE_SCHEDULER_LAMBDA`

- Validates request parameters (`participant_id`, `scheduled_at`)

  

### 6. **Database Validation**

  

- Queries `workflow_action_participant_pivot` table

- Validates that the action/participant combination exists and is not processed

- Creates a `Participant` object for execution

  

### 7. **Action Execution**

  

- Dispatches `ExecuteActionJob` with action and participant

- `ExecuteActionJob` uses cache locking to prevent duplicate executions

- Processes the action based on its type (email, SMS, LINE, etc.)

  

### 8. **Action Type Execution**

  

- Different executors handle different action types:

- `EmailActionExecutor`: Sends emails

- `SendSmsActionExecutor`: Sends SMS

- `SendLineActionExecutor`: Sends LINE messages

- `NotificationActionExecutor`: Sends notifications

- `DelayActionExecutor`: Handles delays

- `ConditionActionExecutor`: Handles conditions

- `AwaitEventExecutor`: Waits for events

  

### 9. **Event Handling**

  

- After execution, `ExecutedEvent` is fired

- `DispatchNextActionListener` handles the event

- `DispatchNextActionJob` processes the next action in the workflow

  

## Key Components

  

### Queue Configuration

  

```php

// config/queue.php

'scheduler-on-demand' => [

'driver' => 'sqs',

'queue' => $queueConfig['queue_name_prefix'].'scheduler-on-demand',

]

```

  

### Feature Flag

  

```php

// app/Models/Core/FeatureFlag.php

public const FEATURE_FLAG_WORKFLOW_ACTION_EXECUTE_SCHEDULER_LAMBDA = 'WORKFLOW_ACTION_EXECUTE_SCHEDULER_LAMBDA';

```

  

### Message Structure

  

```json

{

"code": "workflow_action_execute",

"unique_id": "md5_hash_of_message",

"scheduled_at": "2024-01-01T10:00:00Z",

"payload": {

"action_id": 123,

"participant_id": 456,

"scheduled_at": "2024-01-01 10:00:00"

},

"handler_options": {

"driver": "http",

"headers": {

"Content-Type": "application/json",

"Authorization": "Bearer {client_code}",

"DGM-ACCOUNT-ID": "account_id"

},

"method": "POST",

"url": "https://api.internal/v2/workflow-actions/123/execute"

}

}

```

  

## Error Handling

  

1. **Feature Flag Disabled**: Returns `requires_feature_flag_workflow_action_execute_scheduler_lambda`

2. **Invalid Pivot Data**: Returns `not_found`

3. **Duplicate Execution**: Cache lock prevents multiple executions

4. **Queue Failures**: Retry mechanism with exponential backoff

5. **HTTP Failures**: Lambda retry mechanism for failed API calls