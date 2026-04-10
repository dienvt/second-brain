# Auto query v1

Dính con cron v1 đang phải chạy nhiều instance và đang giải quyết đụng độ bằng cách config port nào chạy cái gì?

  

Solution:

Locking 1 key nào đó. Khi đó chỉ có 1 con chạy được cron? cách này có ổn hay không.

  

xem thử thằng nào đang delay activemq

Đang init reminder delay sau khi xử lý async kafka. Không thể tách kafka vì tách sẽ bị tình trạng bill đã thanh toán nhưng mà nghi nhận là đã pay.

# Check list

Jenkin:

- [x] Moving git repo
    - [x] Create new repo
    - [x] Mirror from old repo
- [x] config folder ( /config)
- [x] new config loading mechanism
- [x] Move activeMQ to file config
- [x] Add ReturnCode table in AdminDB
- [x] Health-check
    - [x] /health : heath check
    - [x] /info: ready to serve check
    - [x] grpc health service
- [x] setup value in nacos ([https://dev-nacos.zalopay.vn/nacos/](https://dev-nacos.zalopay.vn/nacos/))
- [x] Docker file
    - [x] build stage
    - [x] run stage
    - [x] run file
- [x] Migrate db dev
- [x] Monitor
    - [x] expose metrics
- [x] Require domain ([https://confluence.zalopay.vn/x/AzPVAw](https://confluence.zalopay.vn/x/AzPVAw))
- [x] new Kafka async broker (cause bare metal could continue consume data)

  

# Rollout step

- [ ] Remove redisson read
- [ ] New Redis?
- [ ] Switch updateMultiBill to new K8s
    - [ ] Disable Physical
    - [ ] Enable K8s
- [ ] Switch consume auto query to new K8s
    - [ ] Disable Physical
    - [ ] Enable K8s

  

- [ ] Update config off auto query v1 từ từ.
- [ ]

  

  

| rex "delay=(?<delayTimeSecond>.\d+)" | where delayTimeSecond > 0 AND delayTimeSecond < 86400

  

4 ngày 347217

  

1659120480000 + 288797000

  

  

get cache `getZaloPay2CustomerGroup`

- read: getAllCustomer, Get Storage, Check registered, Numbill
- write: put zalopay 2 customer

  

# Check list

- [x] Notification
- [ ] Check config
    - [ ] redis
    - [ ] redis prefix
    - [ ] activemq
    - [ ] kafka
    - [ ] …