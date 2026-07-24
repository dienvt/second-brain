## Daily Usage
Call plan call digima 
call plan call subscription
NameCallPlanCallDigima,
FeatureCallPlanCallDigima
Segment co cai kia 
## Overview
We have table name `feature_usage` in each schema
Then for each row we have structure like this
```
[
  {
    "id": 197,
    "created_at": "2026-02-01 13:31:27",
    "updated_at": "2026-02-09 23:58:39",
    "deleted_at": null,
    "feature": "contacts",
    "from": "2026-01-31 15:00:00",
    "to": "2026-02-28 14:59:59",
    "amount": 731,
    "daily_usage": {"2026-01-31 15:00:00 to 2026-02-01 14:59:59": 586, "2026-02-01 15:00:00 to 2026-02-02 14:59:59": 587, "2026-02-02 15:00:00 to 2026-02-03 14:59:59": 610, "2026-02-03 15:00:00 to 2026-02-04 14:59:59": 655, "2026-02-04 15:00:00 to 2026-02-05 14:59:59": 680, "2026-02-05 15:00:00 to 2026-02-06 14:59:59": 681, "2026-02-06 15:00:00 to 2026-02-07 14:59:59": 682, "2026-02-07 15:00:00 to 2026-02-08 14:59:59": 684, "2026-02-08 15:00:00 to 2026-02-09 14:59:59": 731, "2026-02-09 15:00:00 to 2026-02-10 14:59:59": 731}
  }
]
```
How to know how many usage was missing?
contacts, storage: peak usage
PhoneNumber/CallPlans: the descrease only calculate at the end of month. Build theo month và theo daily.
- Deleted_at >= from
- created_at <= to
- phone created 2026-01-02T00:00:00 -> deleted 2026-01-03T00:00:00
- Usage 2026-01-01: 0
- Usage 2026-01-02: 1
- Usage 2026-01-03: `2026-01-03T00:00:00` >= `2026-01-03T00:00:00` && `2026-01-02T00:00:00` <= `2026-01-03T23:59:59`
- Usage 2026-01-04: `2026-01-03T00:00:00` >= `2026-01-04T00:00:00` && `2026-01-02T00:00:00` <= `2026-01-04T23:59:59`
- Usage 2026-01-04: 1

Why there is filter where: 
```
->where('plan', PlanSubscription::CALL_PLAN_CALL_DIGIMA)
```
call plan tinh peak
started_at, ended_at????

PhoneNumber no peak compare vs CallPlans peak compare


## Crm
Normally
```
return Contact::query()
->notArchivedOrDuplicate()
->where('created_at', '<=', $to);
```
If the begin of the month, We we init the usage with zero then
Get 
usage by month: `preUsage`
* If don't have: init monthly usage with 0 
* `current` usage by: sum all until now (date)
* set monthly usage by max (`current`, `preUsage`)
get usage by date 
* Save usage by date into the schema

## Call plan subscription

```
The project tracks 15 billable features across several categories:
Core Features:
- Contact/UsageManager - Counts non-archived, non-duplicate contacts
- Email/UsageManager - Tracks dispatched email recipients (local + workflow + external)
- Storage/UsageManager - Aggregates storage from local files + external File/Email services
- Lead/UsageManager - Counts lead submissions (local + external Lead service)
Communication:
- Sms/OutboundUsageManager - SMS messages sent (via SMS microservice)
- Sms/InboundUsageManager - SMS messages received (via SMS microservice)
- Line/OutboundUsageManager - LINE messages sent (via LINE microservice)
Phone/Call:
- Phone/Number/UsageManager - Active phone numbers owned
- Phone/Call/Outgoing/Landline/Mobile - Outgoing call durations
- Phone/Call/Incoming/Landline/Mobile - Incoming call durations
- Call/PlanSubscription/UsageManager - Active call plan subscriptions
Web Tracking:
- Web/Tracking/Visit/UsageManager - Web tracking visits
  
I want to deep dive the implementation detail. The usage is monthly right? so what is the difference between those? I know there some usage calculate from the begining to the end but I've known correctly which one?
```




## Summary (MiniMax M2.1)

| Pattern      | Features                            | Calculation                                     | Decreases? | Query Timeframe             |
| ------------ | ----------------------------------- | ----------------------------------------------- | ---------- | --------------------------- |
| Peak Billing | contacts, storage                   | Cumulative from account creation or month start | ❌ No       | Past → Present              |
| Event-Based  | email, calls, sms, line, web, leads | Sum of events within period                     | ✅ Yes      | Within period               |
| Snapshot     | phone_numbers, call_plans           | Count existing at period end                    | ✅ Yes      | Created before → Period end |
Which Calculate from Beginning to End?
Pattern 1 (Peak Billing):
- storage - From account creation → given date (line 110-112)
- contacts - From beginning of month → given date (implicit via peak tracking)
Pattern 2 & 3:
- Calculate only events/state within the specific period (day/month), not cumulative from account creation
Monthly Calculation Workflow
All implementations follow this flow:
1. Normalize date to billing timezone (config('billing.timezone'))
2. Get or create Usage record for the month (via get_month_bounds())
3. Calculate period amounts:
   - Month: startOfMonth → endOfMonth
   - Day: startOfDay → endOfDay
4. Update amount (monthly total)
5. Update daily_usage (JSON array with daily breakdown)
6. Save to MongoDB
The key differences are what data is queried and how the amount is calculated (peak vs. sum vs. snapshot).

## Summary: Billing Usage Calculation Types
### Cumulative (Peak-Based) - 2 Features
These calculate usage from the beginning of time (or account creation) to the current date:

| Feature              | Calculation Logic                                                                                                                                                            |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Contact/UsageManager | Counts all non-archived, non-duplicate contacts created up to the current date (created_at <= $to). Uses peak billing - only updates if current count exceeds stored amount. |
| Storage/UsageManager | Calculates total storage from account creation date to current date. Includes files, emails, and external service storage. Uses peak billing.                                |
|                      |                                                                                                                                                                              |
### Period-Based (Monthly) - 10 Features
These count events/actions that occurred within the billing month period:

| Feature                      | Calculation Logic                                                              |
| ---------------------------- | ------------------------------------------------------------------------------ |
| Email/UsageManager           | Dispatched email events whereBetween('occurred_at', [$from, $to])              |
| Lead/UsageManager            | Lead messages where('created_at', '>=', $from)->where('created_at', '<=', $to) |
| Sms/OutboundUsageManager     | External API call with from/to date parameters                                 |
| Sms/InboundUsageManager      | External API call with from/to date parameters                                 |
| Line/OutboundUsageManager    | External API call with period parameters                                       |
| Phone/Call/Outgoing/Landline | Sum of call duration where ended_at is within period                           |
| Phone/Call/Outgoing/Mobile   | Sum of call duration where ended_at is within period                           |
| Phone/Call/Incoming/Landline | Sum of call duration where ended_at is within period                           |
| Phone/Call/Incoming/Mobile   | Sum of call duration where ended_at is within period                           |
| Web/Tracking/Visit           | Visits where('created_at', '>=', $from)->where('created_at', '<=', $to)        |
|                              |                                                                                |
### Period-Based (Point-in-Time) - 2 Features
These count resources that existed/were active during the billing period:

| Feature                            | Calculation Logic                                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Phone/Number/UsageManager          | Phone numbers where created_at <= $to AND (deleted_at IS NULL OR deleted_at >= $from)                                          |
| Call/PlanSubscription/UsageManager | Distinct subscribers with active plans where started_at <= $to AND (ended_at IS NULL OR ended_at >= $from). Uses peak billing. |
### Key Differences Explained
1. Cumulative (Contact, Storage): These track the maximum count ever reached. If you have 100 contacts today, delete 20 tomorrow, you're still billed for 100 (peak). Storage works similarly.
2. Period-Based Monthly (Email, SMS, Lead, Calls, etc.): These reset each month. If you send 500 emails in January and 300 in February, you're billed for 500 and 300 respectively.
3. Period-Based Point-in-Time (Phone Numbers, Call Plans): These count resources that overlapped with the billing period. A phone number purchased mid-month still counts. Call Plan Subscription also uses peak billing.


## Complete Billing Usage Implementation Details
### Overview: Trigger Mechanism
All usage managers share the same trigger pattern:
1. Daily Scheduled Job - UpsertDailyFeaturesUsageScheduler runs at 00:10 Asia/Tokyo for the previous day
2. Commands - digima:feature-usage-all-upsert-dispatch and digima:feature-usage-all-upsert
3. Jobs - CalculateFeatureUsageJob (queued) and CalculateUsageJob (sync)
4. Account Status Change - UpdateAccountJob triggers when account becomes disabled/cancelled
5. Admin API - POST /admin/v2/accounts/{account}/feature/usage/recalculate
Important: There are NO real-time event listeners that trigger usage calculation when resources are created/updated. Usage is calculated retrospectively on a schedule.

### Detailed Feature Breakdown
#### 1. Contact/UsageManager - CUMULATIVE (Peak)
| Aspect | Details |
|--------|---------|
| File | app/Services/Contact/UsageManager.php |
| Feature Property | contacts |
| Data Source | Local database only |
| Query | Contact::notArchivedOrDuplicate()->where('created_at', '<=', $to) |
Query Conditions:
```
// notArchivedOrDuplicate scope:
whereNull('archived_at')->orWhereHas('duplicatePivot')
// PLUS
where('created_at', '<=', $to)
```
What's Counted:
- ✅ Active contacts (not archived)
- ✅ Duplicate contacts (archived but has DuplicatePivot record)
- ❌ Regular archived contacts
Billing Type: Peak billing - only updates if current > stored
---
#### 2. Email/UsageManager - PERIOD-BASED (Monthly)
| Aspect           | Details                                  |
| ---------------- | ---------------------------------------- |
| File             | app/Services/Email/UsageManager.php      |
| Feature Property | email                                    |
| Data Sources     | Local DB + External Email Service (gRPC) |
Query Conditions (3 sources combined):
```
// Source 1: Local email dispatch events
Event::where('name', 'dispatched')
     ->whereBetween('occurred_at', [$from, $to])
     ->select('recipient_id')->distinct()->count()
// Source 2: Workflow notification recipients
workflow_action_participant_pivot JOIN workflow_actions
WHERE type = 'notification' AND processed_at BETWEEN $from AND $to
// Source 3: External Email Inbox (gRPC)
FeatureUsage->show($from, $to)->getEnvelopes()->getResourceCount()

??? lam gi o day

```

```

```

What's Counted:
- ✅ Distinct email recipients dispatched within period
- ✅ Workflow notification participants processed within period
- ✅ External email inbox envelopes (if subscription active)
---
#### 3. Storage/UsageManager - CUMULATIVE (Peak)
| Aspect           | Details                                          |
| ---------------- | ------------------------------------------------ |
| File             | app/Services/Storage/UsageManager.php            |
| Feature Property | storage                                          |
| Data Sources     | Local DB + External File & Email Services (gRPC) |
| Period           | From account creation date to current date       |
Query Conditions (4 sources combined):
```
// Source 1: Local files (sum of size)
// Billable File Types: Only email_attachment and call_recording
File::withTrashed()
    ->where('created_at', '<=', $to)
    ->whereIn('type', ['email_attachment', 'call_recording'])
    ->where(fn($q) => $q->whereNull('deleted_at')->orWhere('deleted_at', '>=', $from))
    ->sum('size')


// Source 2: Local emails (text length sum)
Email::where('created_at', '<=', $to)
     ->selectRaw('SUM(LENGTH(subject)+LENGTH(body_text)+LENGTH(body_html))')

// Source 3: External Email Inbox (gRPC)
envelopes.storageBytes + attachments.storageBytes

// Source 4: External File Management (gRPC)
sharedFiles.storageBytes + files.storageBytes
```
Billing Type: Peak billing - only updates if current > stored

---
#### 4. Lead/UsageManager - PERIOD-BASED (Monthly)
| Aspect | Details |
|--------|---------|
| File | app/Services/Lead/UsageManager.php |
| Feature Property | lead_acquisition |
| Data Sources | Local DB + External Lead Service (gRPC) |
Query Conditions (2 sources combined):
```
// Source 1: Local lead inbox messages
Message::where('created_at', '>=', $from)
       ->where('created_at', '<=', $to)
       ->whereNull('failed_reason')
       ->count()
// Source 2: External Lead Service (gRPC)
Submissions->list(filters: {
    'created_at_from': $from,
    'created_at_to': $to,
    'is_failed': false
})->getTotal()
```
What's Excluded:
- ❌ Messages with failed_reason (invalid, missing_parsed_data, deduplicated, validation_failed)
- ❌ Failed submissions from external service
---
#### 5. SMS/OutboundUsageManager - PERIOD-BASED (Monthly)
| Aspect | Details |
|--------|---------|
| File | app/Services/Sms/OutboundUsageManager.php |
| Feature Property | sms_outbound_messages |
| Data Source | External SMS Service (REST API) |
API Call:
`GET {DOMAIN_SMS_API}/v2/statistics?from={Y-m-d H:i:s}&to={Y-m-d H:i:s}`
Headers:
  Authorization: base64({account_id, account_code})
Response field used: `total_outbound_sent_parts`
What's Counted: SMS parts/segments sent (long messages split into multiple parts)




---
#### 6. SMS/InboundUsageManager - PERIOD-BASED (Monthly)
| Aspect           | Details                                  |
| ---------------- | ---------------------------------------- |
| File             | app/Services/Sms/InboundUsageManager.php |
| Feature Property | sms_inbound_messages                     |
| Data Source      | External SMS Service (REST API)          |
API Call: Same endpoint as outbound
Response field used: `total_inbound_received`
```
total_inbound_received = count of sms_statistics documents with event_type = "message_inbound_received" matching the given carrier/date-range/message_id
  filters, computed on-demand at query time (via domain/services/statistic/service.go:25 Aggregate), exposed through the HTTP v2/v3 and gRPC statistic
  endpoints (interface/resources/statistic/.../controller.go) and also used per-message in interface/resources/message/http/v2|v3/transformers/message.go.
```

---
#### 7. Line/OutboundUsageManager - PERIOD-BASED (Monthly)
| Aspect           | Details                                    |
| ---------------- | ------------------------------------------ |
| File             | app/Services/Line/OutboundUsageManager.php |
| Feature Property | line_outbound_messages                     |
| Data Source      | External LINE Service (gRPC)               |
API Call:
```
FeatureUsage->show(
    retrieve_from: Timestamp($from),
    retrieve_to: Timestamp($to)
)->getOutboundMessagesCount()
```

In line service:

Calculation (domain/services/feature_usage/service.go:23)

  For the time window [from, to], the service returns a models.FeatureUsage with:
  - **OutboundMessagesCount** = sum of two counts (see below)
  - **ChannelsCount** = hardcoded 0 (not used yet — service.go:37)
  - From, To = the requested window
  - CalculatedAt = time.Now() at call time

  **OutboundMessagesCount** (the billing count) is computed in countMessagesForBilling:
  billingCount = CountSingleMessages(outbound) + CountGroupedMessages(outbound)
  Both filter by exchange_type = outbound.

  Counting rules (infra/database/repositories/mysql/contact_message/repository.go:206)

  Both calls go through countByCondition against contact_messages:
  - db.Unscoped() — soft-deleted rows are still counted
  - occurred_at is within [from, to] inclusive
  - exchange_type = 'outbound'
  - Group filter via getGroupCondition at repository.go:238:
    - Single messages (BelongsToGroupMessage=false): **group_message_recipient_id IS NULL**
    - Grouped messages (BelongsToGroupMessage=true): **group_message_recipient_id IS NOT NULL**, then GROUP BY group_message_recipient_id — so each group blast counts as 1, regardless of how many recipients it had


---
#### 8. Phone/Number/UsageManager - PERIOD-BASED (Point-in-Time)
| Aspect           | Details                                    |
| ---------------- | ------------------------------------------ |
| File             | app/Services/Phone/Number/UsageManager.php |
| Feature Property | call_phone_number                          |
| Data Source      | Local database only                        |
Query Conditions:
```
Number::withTrashed()
      ->where('created_at', '<=', $to)
      ->where(fn($q) => 
          $q->whereNull('deleted_at')
            ->orWhere('deleted_at', '>=', $from)
      )->count()
```
What's Counted: Phone numbers that existed at any point during the period

---
#### 9-12. Phone Call UsageManagers - PERIOD-BASED (Monthly)
| Manager           | Feature Property           | Filter            |
| ----------------- | -------------------------- | ----------------- |
| Outgoing/Landline | call_rate_landline         | is_mobile = false |
| Outgoing/Mobile   | call_rate_mobile           | is_mobile = true  |
| Incoming/Landline | call_rate_landline_forward | is_mobile = false |
| Incoming/Mobile   | call_rate_mobile_forward   | is_mobile = true  |
Common Query Pattern (Outgoing):
```
OutgoingCall::withTrashed()
    ->where('ended_at', '>=', $from)
    ->where('ended_at', '<=', $to)
    ->where('type', 'browser')          // Outgoing only
    ->where('status', '!=', 'failed') ??? 
    ->where('is_mobile', true/false)
    ->where(fn($q) => $q->whereNull('deleted_at')->orWhere('deleted_at', '>=', $from))
Duration Calculation:
// Rounded UP to nearest minute
(CASE WHEN duration % 60 = 0 
 THEN ROUND(duration/60) 
 ELSE ROUND(duration/60+0.5, 0) END) AS duration_in_minutes
```
What's Counted: Sum of call duration in minutes (rounded up)


```
IncomingCall::query()
       ->withTrashed()
       ->where('ended_at', '>=', $from)
       ->where('ended_at', '<=', $to)
       ->where('status', '!=', IncomingCall::STATUS_FAILED)
       ->where('is_mobile', false)
       ->where(
           function (Builder $query) use ($from)
           {
               $query->whereNull('deleted_at')
                     ->orWhere('deleted_at', '>=', $from);
           }
       );
```

---
#### 13. Call/PlanSubscription/UsageManager - PERIOD-BASED (Peak)
| Aspect           | Details                                             |
| ---------------- | --------------------------------------------------- |
| File             | app/Services/Call/PlanSubscription/UsageManager.php |
| Feature Property | call_plan_call_digima                               |
| Data Source      | Local database only                                 |
Query Conditions:
```
PlanSubscription::withTrashed()
    ->where('plan', 'call_digima')
    ->where('started_at', '<=', $to)
    ->where(fn($q) => $q->whereNull('ended_at')->orWhere('ended_at', '>=', $from))
    ->where(fn($q) => $q->whereNull('deleted_at')->orWhere('deleted_at', '>=', $from))
    ->select('subscriber_id')->distinct()->count('subscriber_id')
```
What's Counted: Distinct subscribers with active Call Digima plans during the period
Billing Type: Peak billing - only updates if current > stored

---
#### 14. Web/Tracking/Visit/UsageManager - PERIOD-BASED (Monthly)
| Aspect | Details |
|--------|---------|
| File | app/Services/Web/Tracking/Visit/UsageManager.php |
| Feature Property | web |
| Data Source | Local database only |
Query Conditions:
```
Visit::withTrashed()
     ->where('created_at', '>=', $from)
     ->where('created_at', '<=', $to)
     ->where(fn($q) => $q->whereNull('deleted_at')->orWhere('deleted_at', '>=', $from))
     ->count()
```
What's Counted: Visits created within the period

---
#### Summary Table

| Feature                | Type                    | Peak Billing? | Data Sources                   | Correctly Billing |
| ---------------------- | ----------------------- | ------------- | ------------------------------ | ----------------- |
| Contact                | Cumulative              | ✅ Yes         | Local                          | ✅                 |
| Storage                | Cumulative              | ✅ Yes         | Local + External (File, Email) |                   |
| Email                  | Period                  | ❌ No          | Local + External (Email)       |                   |
| Lead                   | Period                  | ❌ No          | Local + External (Lead)        |                   |
| SMS Outbound           | Period                  | ❌ No          | External (SMS)                 |                   |
| SMS Inbound            | Period                  | ❌ No          | External (SMS)                 |                   |
| LINE Outbound          | Period                  | ❌ No          | External (LINE)                |                   |
| Phone Number           | Period (Point-in-time)  | ❌ No          | Local                          |                   |
| Call Outgoing Landline | Period                  | ❌ No          | Local                          |                   |
| Call Outgoing Mobile   | Period                  | ❌ No          | Local                          |                   |
| Call Incoming Landline | Period                  | ❌ No          | Local                          |                   |
| Call Incoming Mobile   | Period                  | ❌ No          | Local                          |                   |
| Call Plan Subscription | Period  (Point-in-time) | ❌ No          | Local                          |                   |
| Web Tracking Visit     | Period                  | ❌ No          | Local                          |                   |

---

Cai peak, accumulated v
