### Local
`failover:(tcp://10.109.3.26:61616)?jms.prefetchPolicy.queuePrefetch=5`

###  Production

| service  | activeMQ         | Admin                                                         |
| -------- | ---------------- | ------------------------------------------------------------- |
| core-api | tcp://10.50.49.75:8045 | http://10.50.49.75:7045 (`admin` / `youradminactivepassword`) |
| tracking | tcp://10.50.49.75:8049 | http://10.50.49.75:7049 (`admin` / `youradminactivepassword`)                                               |

---
