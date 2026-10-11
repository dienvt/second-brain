## Overview: Trigger Mechanism
All usage managers share the same trigger pattern:
1. Daily Scheduled Job - UpsertDailyFeaturesUsageScheduler runs at 00:10 Asia/Tokyo for the previous day
2. Commands - digima:feature-usage-all-upsert-dispatch and digima:feature-usage-all-upsert
3. Jobs - CalculateFeatureUsageJob (queued) and CalculateUsageJob (sync)
4. Account Status Change - UpdateAccountJob triggers when account becomes disabled/cancelled
5. Admin API - POST /admin/v2/accounts/{account}/feature/usage/recalculate
Important: There are NO real-time event listeners that trigger usage calculation when resources are created/updated. Usage is calculated retrospectively on a schedule.

## Detailed Feature Breakdown
### 1. Contact/UsageManager - CUMULATIVE (Peak)
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
### 2. Email/UsageManager - PERIOD-BASED (Monthly)
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
### 3. Storage/UsageManager - CUMULATIVE (Peak)
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
### 4. Lead/UsageManager - PERIOD-BASED (Monthly)
| Aspect           | Details                                 |
| ---------------- | --------------------------------------- |
| File             | app/Services/Lead/UsageManager.php      |
| Feature Property | lead_acquisition                        |
| Data Sources     | Local DB + External Lead Service (gRPC) |
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
### 5. SMS/OutboundUsageManager - PERIOD-BASED (Monthly)
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
### 6. SMS/InboundUsageManager - PERIOD-BASED (Monthly)
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
### 7. Line/OutboundUsageManager - PERIOD-BASED (Monthly)
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
### 8. Phone/Number/UsageManager - PERIOD-BASED (Point-in-Time)
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
### 9-12. Phone Call UsageManagers - PERIOD-BASED (Monthly)
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
### 13. Call/PlanSubscription/UsageManager - PERIOD-BASED (Peak)
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
### 14. Web/Tracking/Visit/UsageManager - PERIOD-BASED (Monthly)
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
