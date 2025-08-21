## Insight
```
fields @message, time_elapsed
| filter @message like "Request process finished"
| sort time_elapsed desc
| limit 100
```