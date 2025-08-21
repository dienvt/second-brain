---
tags:
  - rdbms
---
## Select For UPDATE
Lock the record you select. 
When another process try to **READ/UPDATE** the locked record, the process will be block. So that those will be work correctly
```sql
SELECT count(1) AS total FROM user_tickets WHERE user_id = ?; -- the simultaneously process will be blocked here
-- If total < 10
INSERT INTO user_tickets VALUES(?,?);
```



## SELECT FOR UPDATE SKIP LOCKED
nhưng mà con outbox dispatcher nếu scale lên >= 2 pods thì sẽ bị trùng message. Nên là dùng:
Select * from events where status='PENDING' FOR UPDATE SKIP LOCKED LIMIT 0,10; 
để fetch events