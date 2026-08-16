Your order processing service runs on SQS. Normal load: 200 orders/min. Consumers keep up fine. Then Black Friday hits. Producers start pushing 4,000 orders/min. Queue depth climbs to 80,000 messages
in 20 minutes. Your downstream DB is at 95% CPU. Consumers are falling behind and you're watching the
queue grow in real time. You need to handle this backpressure. What do you do?
**A)** Scale consumers horizontally — add more Lambda functions / EC2 workers to chew through the backlog faster.
**B)** Set a visibility timeout and route failures to a dead-letter queue to protect against poison pills.
**C)** Rate-limit producers at the source — use a token bucket or sliding window to cap how fast messages enter the queue.
**D)** Switch to SQS delay queues — defer message visibility to spread out delivery and reduce consumer
pressure.