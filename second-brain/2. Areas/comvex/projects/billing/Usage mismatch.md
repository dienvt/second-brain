## Todo List

nanoid, cross-spawn, ws, tar, follow-redirects, store2, path-to-regexp 3.0→3.3
 * Storage over bill
	* Sotrage size matchd the feature usage table?
	* Ghost attachment in storage
	* Storage segment
	* ghost attachments or not
* Check ghost attachment
* Move updated billing and backend-app to new card
* Create card for the ghost attachment mismatch.

- Ok with backfill then migrate account pricing plan
- Create sync mechasim
- We have to to fix the attachment created as well. Let publish only attach_resources created and in billing we still keep idempotency with the attachment_id. (because one attachment can link to many envelope)
- Show the new billing app. 
- 21st July
- There might bee many event usage in the day end from 59.000 to 59.999 but monthly will correct so ? 
- 

In Billing: Is create segment skip dulication?
```

  Two consequences worth knowing:

  1. Duplicated segments double-count on the invoice, since invoice creation pulls segments via GetByAccountAndPeriod, which returns all overlapping documents.
  2. Even with Recalculate=true there's a race: UpsertDaily protects its find-then-save with a distributed lock (service.go:231), but Create has no lock, so two concurrent create jobs can both get NotFound and both insert.

  So: dedup only happens when the caller explicitly passes recalculate=true, and it's find-and-update, not a real upsert or unique constraint.
```

Backend-app: What happend if there is no pricing_plan is set in account
```
⏺ What happens when pricing_plan is NULL on an account

  Schema: pricing_plan is a nullable string(100) column (added in 2025_03_05_083000_core_alter_accounts_table7.php), no DB default. There's no enum/cast — just a raw string, so reading null never throws. New accounts created via CreateAccountJob don't set it at all, so they get NULL unless explicitly assigned.

  Two different behaviors depending on where you look:

  1. Billing property lookup — Account::billingPropertiesForFeature() (app/Models/Core/Account.php:348):
  $feature = $this->pricing_plan ?: Account::BILLING_PROPERTY_PRICING_PLAN_DEFAULT;
  2. Null is explicitly coalesced to 'pricing_plan_default', then it looks up that key in the account's billing_properties, falling back to the default_billing_properties setting. This is used by AbstractSegmentMaker::initialize() and by your BackfillAccountBillingPropertiesCommand (line 214) — same pattern, both
  intentionally treat null as pricing_plan_default.
  3. Segment/invoice generation — AbstractSegmentMaker::make() (line 207-209):
  if (! $this->account->pricing_plan) { return null; }
  // "It won't create billing segment for pricing plan if account has no pricing plan."
  4. This is a different, un-defaulted check. So even though initialize() just resolved cost/tiers as if the account were on pricing_plan_default, make() throws that away and skips creating any pricing-plan billing segment entirely when the raw column is NULL.

  Net effect: a NULL pricing_plan account never gets billed for the pricing-plan feature at all (no segment created), whereas an account explicitly set to 'pricing_plan_default' would get a segment (cost 0, since that's the free-tier default). Null and pricing_plan_default look equivalent everywhere property-lookup
  code runs, but they diverge specifically at segment/invoice creation — this is the same "quietly different code path for missing/null value" shape as the billing_properties issue you found earlier, just isolated to one method (make()) rather than being a systemic gap.

  UpdateAccountJob::refineBillingProperties() and AccountTransformer just pass null through harmlessly (no plan-prefixed keys get refined/kept; API returns "pricing_plan": null).

  So: it's intentional, not a crash risk, but worth knowing — if you ever want null-plan accounts to be billed as pricing_plan_default consistently, AbstractSegmentMaker::make() line 207-209 is the one place that would need to change.

```

If this is no account's pricing plan it mean no calculate billing?
-> yes



- Cai phone number thuc su la khac do
	- thuc su la lay startofday vao start of month
```
return Number::query()
->withTrashed()
->where('created_at', '<=', $to)
->where(
function (Builder $query) use ($from)
{
$query->whereNull('deleted_at')
->orWhere('deleted_at', '>=', $from);
}
);
```
- 

- Xoa segment roi trigger tao lai? vay thi phai them nhung feature khac
- [ ] Miss 2 flow sync billing properties va sms segment, still need to sync
- [ ] 
- 

## Release Steps
- Chay daily upsert -> value sync to the billing.
- Deploy ca billing and backend-app
- run db bootrap in billing
```
db:bootstrap all
```
* nếu force phải run lại migrate default
```
digima:billing-default-properties-sync 1 --force
```
- Run backfill in backend-app: 
```
digima:accounts-backfill-billing-properties 
```
- Run billing properties sync again 
```
digima-tmp:billing-properties-migrate all
```
- 
- Regenerate June Invoice:
- Sync sms_phone_number
```

```
- Trigger daily segment calculation in June
```

```

## Scheduler
Schedule
1h : reconciliation
4h: Daily segment 
Monthly Invoice: 6h
19h: Carry over

Has to run sync sms phone number before the invoice generation start


* Backend-app accumulative usage:
```
Resume this session with:
claude --resume 495f3805-00cf-471b-bc8d-d72b5080bbfb
```

* Default billing properties
* 

| date       | Task                                   | status | Fix applied after |                                                                                                                                                                                                          |
| ---------- | -------------------------------------- | ------ | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|            | Fix Call made event in Backend-App     | 🟡🔴   |                   |                                                                                                                                                                                                          |
|            | [Billing] Fix the duration calculation | 🟢🟢   |                   |                                                                                                                                                                                                          |
|            | [BackendApp] Fix the phone made event  | 🟡🔴   |                   |                                                                                                                                                                                                          |
|            | Investigate line_message               | 🟢🟢   | 2026-05-23:00:00  | Duplicate idempotency key. The contact_group_recipient_id                                                                                                                                                |
|            | Investigate call_phone_number          | 🟡🔴   |                   | carry over error                                                                                                                                                                                         |
|            | Investigate call_plan_call_digima      | 🟡🔴   |                   | carry over error                                                                                                                                                                                         |
|            | Investigate web_tracking_visit         | 🟢🟢   |                   | Maybe there is some duplicate webtracingvisit. Because it upsert so it still publish twice                                                                                                               |
|            | Inviestigate sms_message_outbound      | 🟢🟢   |                   | The events duplicated                                                                                                                                                                                    |
|            | Inviestigate email_message             | 🟢🔴   |                   | There is mismatch in idempotency key. So we have to include the contact ID as well<br>In case workflow: if a message sent to a contact twice. <br>Biliing count it one but backend-app count it two.<br> |
|            |                                        |        |                   |                                                                                                                                                                                                          |
|            | call_rate_landline_incoming            |        |                   | Ko nhan dc event                                                                                                                                                                                         |
|            |                                        |        |                   |                                                                                                                                                                                                          |
| 2026-07-10 | `call_rate_landline_outgoing`          | 🟢🟢   |                   | Some call updated to 0? busy                                                                                                                                                                             |
| 2026-07-10 | `email_message`                        | 🟢🟢   |                   | duplicate `recipient_id` in `email_envelope_recipient_events`                                                                                                                                            |
| 2026-07-12 | `storage_size`                         | 🟢🔴   |                   | File API filtered by type which only applied for backend-app only.                                                                                                                                       |
| 2026-07-10 | `call_plan_digima_call`                | 🟢🟢   |                   |                                                                                                                                                                                                          |
| 07-16      | `sms_message_inbound`                  | 🟢🔴   |                   | Fix in sms-api time bound.<br>The backend-app request to 59 second<br>                                                                                                                                   |
| 07-16      | `call_rate_landline_outgoing`          | 🟢🔴   |                   | Sometime we update the status from completed to failed<br>Sometime we update the duration                                                                                                                |
|            |                                        |        |                   |                                                                                                                                                                                                          |

* [ ] migrate usage?
* [ ] But we have to clean the usage right? 
* [ ] `digima-tmp:usage-migrate all --daily --to="2026-05-10" --force`
* [ ] Daily trigger
* [ ] Trigger daily segment
- Send rabbitmq
- routing key: `v1.segments.upsert_daily_trigger`
- payload 
```
{"accounts":{"all":true},"from":"2021-01-01T00:00:00+09:00","to":"2021-01-31T23:59:59.99999999+09:00"}
```

## Query
```
fields @timestamp, @message
| filter msg like "Usage reconciliation mismatch - creating correction record" AND feature NOT LIKE "crm_contact" AND feature NOT LIKE "storage_size" AND feature NOT LIKE "sms_message_inbound" AND feature NOT LIKE "sms_message_outbound" AND feature NOT LIKE "line_message"
| sort @timestamp desc
| limit 10000
```


"crm_contact" AND feature NOT LIKE "storage_size" AND feature NOT LIKE "sms_message_inbound" AND feature NOT LIKE "sms_message_outbound" AND feature NOT LIKE "line_message"



| Feature                     | Backend-app                                | billing                                 |                                                          |
| --------------------------- | ------------------------------------------ | --------------------------------------- | -------------------------------------------------------- |
| call_rate_landline_outgoing | reconcile in minutes. Sum of each ceilling | reconcile in second. ceiling of all sum | van mismatch?                                            |
| call_rate_landline_incoming | reconcile in minutes                       | reconcile in second                     | 🟡 why it positive when minute always lower than second? |
| crm_contact                 | accumulate                                 |                                         | ❌                                                        |
| storage_size                | accumulate                                 |                                         | ❌                                                        |
| sms_message_inbound         |                                            |                                         | ✅                                                        |
| sms_message_outbound        |                                            |                                         | ✅                                                        |
| line_message                |                                            |                                         | ❌                                                        |
| call_phone_number           | accumulate                                 |                                         | ❌                                                        |
| call_plan_call_digima       | accumulate                                 |                                         | ❌                                                        |
| web_tracking_visit          |                                            | on                                      | ❌                                                        |


```

 Branched conversation. You are now in the new branch (session 8bc73368-3907-416b-bb66-0ff3c9f51144). Use /resume fa192b14-5dcc-489b-8044-5ec20c3bcec1 to return to the original, or run
      claude -r fa192b14-5dcc-489b-8044-5ec20c3bcec1 in a new terminal.

```

```
  ⎿  Branched conversation. You are now in the new branch (session 529834a5-09e2-49da-aebd-df63511f957d). Use /resume 8bc73368-3907-416b-bb66-0ff3c9f51144 ("when was record inserted into phone_outgoing_calls? (Branch)") to return to the original, or run claude -r 8bc73368-3907-416b-bb66-0ff3c9f51144 in a new terminal.
```