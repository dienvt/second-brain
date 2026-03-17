Search for reconcile:
```
{$.msg="Correction usage record created" && $.feature = "storage_size" && $.account_id = 97}
```

**Note:**
- Decided to store full accumulated usage:
	- Don't need start balance???? Yes
	- Stop carry over job, now we fix the accumulated by reconcilition job
	- The reconcilition will create usage at the begin of the month
- How we migrate the current month usages:
	- The milestone time is `2026-03-03T15:00:00Z` (`2026-03-04T00:00:00+09:00`)
	- Whe have to delete usage before that milestone
- Jan, Feb is ????

## Local test

Account 2 :
Migrate 2025 Jan to 2025 Dec
- Execute: 
```
docker exec app-api php artisan digima-tmp:usage-migrate 2 --daily --to=2021-01-31T14:59:59Z
docker exec app-api php artisan digima-tmp:usage-migrate 2 --daily --to=2021-02-28T14:59:59Z
```
Trigger daily segment
- Send rabbitmq
- routing key: `v1.segments.upsert_daily_trigger`
- payload
```
{"accounts":{"all":true},"from":"2021-02-01T00:00:00+09:00","to":"2021-02-28T14:59:59.99999999+09:00"}
```
Trigger monthly invoice
routing key
```
v1.account_pricing_plans.calculate
```
payload
```
{"accounts":{"all":false,"ids":["499"]},"from":"2025-12-01T00:00:00+09:00","to":"2025-12-31T23:59:59.99999999+09:00"}
```
