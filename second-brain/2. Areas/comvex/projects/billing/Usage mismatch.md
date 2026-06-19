## Todo List

Schedule
1h : reconciliation
4h: Daily segment 
Monthly Invoice: 6h
19h: Carry over

* Backend-app accumulative usage:
```
Resume this session with:
claude --resume 495f3805-00cf-471b-bc8d-d72b5080bbfb
```

* Default billing properties
* 

| Task                                   | status | Fix applied after |                                                                                                                                                                                                      |
| -------------------------------------- | ------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Fix Call made event in Backend-App     | 🟡🔴   |                   |                                                                                                                                                                                                      |
| [Billing] Fix the duration calculation | 🟢🟢   |                   |                                                                                                                                                                                                      |
| [BackendApp] Fix the phone made event  | 🟡🔴   |                   |                                                                                                                                                                                                      |
| Investigate line_message               | 🟢🟢   | 2026-05-23:00:00  | Duplicate idempotency key. The contact_group_recipient_id                                                                                                                                            |
| Investigate call_phone_number          | 🟡🔴   |                   | carry over error                                                                                                                                                                                     |
| Investigate call_plan_call_digima      | 🟡🔴   |                   | carry over error                                                                                                                                                                                     |
| Investigate web_tracking_visit         | 🟢🟢   |                   | Maybe there is some duplicate webtracingvisit. Because it upsert so it still publish twice                                                                                                           |
| Inviestigate sms_message_outbound      | 🟢🟢   |                   | The events duplicated                                                                                                                                                                                |
| Inviestigate email_message             | 🟢🔴   |                   | There is mismatch in idempotency key. So we have to include the contact ID as well<br>In case workflow: if a message sent to a contact twice. <br>Biliing count it one but backend-app count it two. |
|                                        |        |                   |                                                                                                                                                                                                      |
| call_rate_landline_incoming            |        |                   | Ko nhan dc event                                                                                                                                                                                     |
|                                        |        |                   |                                                                                                                                                                                                      |

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