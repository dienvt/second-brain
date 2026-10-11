Clone previous assessment sheet

run query

```
-- '2026-08-31 15:00:00'
-- '2026-09-30 14:59:59'
SELECT account_id, SUM(discounted_cost) as discounted_cost  
FROM billing_segments  
WHERE from_date >= :from_date and from_date < :to_date  
GROUP BY account_id  
ORDER BY account_id;
```

Export from billing

