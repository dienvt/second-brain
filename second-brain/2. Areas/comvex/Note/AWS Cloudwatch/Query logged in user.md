```
fields variables.username, ip_address , @timestamp
| filter msg = 'Request process started' and route.0="mutation.login"
| sort @timestamp desc
| limit 10000
```