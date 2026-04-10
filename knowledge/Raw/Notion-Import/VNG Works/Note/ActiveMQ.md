### Local

Bill: `failover:(tcp://10.109.3.26:61616)?jms.prefetchPolicy.queuePrefetch=5`

Telco: `failover:(tcp://10.109.3.50:61616)?jms.prefetchPolicy.queuePrefetch=1`

### Telco

Airtime: 10.50.49.15:7561

3G/4G: 10.50.49.15:7661

Mobilecard: 10.50.49.15:7761

Postpaid: 10.50.49.15:8361

card store: 10.50.49.15:8161

  

Unknown

card store: 10.50.49.15:8161  
Khum bit: 10.50.49.15:8261  

`admin` / `your_password`

### Bill

Tracking: 10.50.49.15:62616 - [https://monitor-amq1-billing.zpapps.vn](https://monitor-amq1-billing.zpapps.vn/) (`admin` / `youradminactivepassword`)

→ 10.50.49.75:8049

  

Core: 10.50.49.16:61616 - [https://monitor-amq-billing.zpapps.vn](https://monitor-amq-billing.zpapps.vn/) (`admin` / `youradminactivepassword`)

→ 10.50.49.75:8045

  

---

New ActiveMQ

10.50.49.73:8046

10.50.49.73:8047

10.50.49.73:8048

10.50.49.73:8049

10.50.49.73:8050

  

10.50.49.75:8046

10.50.49.75:8047

10.50.49.75:8048

10.50.49.75:8049

10.50.49.75:8050

  

---

  

restart bill-tracking-worker

→ update new activemq,

→ push lại thì bị filter đầu billTracking