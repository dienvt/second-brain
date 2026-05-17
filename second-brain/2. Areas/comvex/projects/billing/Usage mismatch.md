## Query
```
fields @timestamp, @message
| filter msg like "Usage reconciliation mismatch - creating correction record" AND feature NOT LIKE "crm_contact" AND feature NOT LIKE "storage_size" AND feature NOT LIKE "sms_message_inbound" AND feature NOT LIKE "sms_message_outbound" AND feature NOT LIKE "line_message"
| sort @timestamp desc
| limit 10000
```
