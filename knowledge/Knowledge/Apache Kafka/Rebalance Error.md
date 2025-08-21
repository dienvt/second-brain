## Symptom

```Plain
[ INFO] [AbstractCoordinator.java ] Attempt to heart beat failed since member id is not valid, reset it and try to re-join group.
[ERROR] [ConsumerCoordinator.java ] Error UNKNOWN_MEMBER_ID occurred while committing offsets for group group_dev_telco_maintenance
[ WARN] [ConsumerCoordinator.java ] Auto offset commit failed:
```

OR

```Plain
[ INFO] [AppTelcoProviderBuz.java ] autoMaintenance end
[ INFO] [AbstractCoordinator.java ] Attempt to heart beat failed since member id is not valid, reset it and try to re-join group.
```

## Cause

- Maybe execute message take to long so heart beat can't come to server in valid time
- Poll() using for heat beat so that when call poll before commit can cause error `member id is not valid`

## Solution

- Increase `session.timeout.ms`
- Increate `consumer.poll(TIMEOUT)` to larger `session.timeout.ms`

## Reference

[java - Error UNKNOWN_MEMBER_ID occurred while committing offsets for group xxx - Stack Overflow](https://stackoverflow.com/a/38972314)