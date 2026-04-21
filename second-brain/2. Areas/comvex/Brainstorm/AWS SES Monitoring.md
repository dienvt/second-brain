# AWS SES Monitoring

## Context

Được đề cập trong cuộc thảo luận về việc cải thiện hệ thống notifications của Digima — thay thế Sendgrid bằng SES cho transactional emails gửi đến Digima users (không phải contacts).

**Lý do chuyển sang SES:**
- Sendgrid đắt hơn SES ~10 lần
- Dedicated IP pool & tenant management qua subuser (lý do dùng Sendgrid) không cần thiết cho notifications
- SES đủ dùng cho internal user notifications

---

## Các phương pháp monitoring

### 1. SES Console
- Dashboard real-time: quota, bounces, complaints, rejects
- Granularity: account-level

### 2. Virtual Deliverability Manager (VDM)
- Dashboard toàn diện nhất
- Track: delivered, bounces (permanent & transient), complaints, opens, clicks
- Có advisor gợi ý cải thiện deliverability
- Granularity: account, ISP, sending identity, configuration set

### 3. SES API — `GetSendStatistics`
- Trả về count: deliveries / bounces / complaints / rejects
- Granularity: account-wide

### 4. Amazon CloudWatch
- Metrics: sends, deliveries, opens, clicks, bounce rate, complaint rate, blacklisted IPs

### 5. Feedback Notifications (SNS hoặc Email)
- Nhận notification khi có bounce / complaint / delivery
- Cách đơn giản nhất để implement webhook nhận bounce events

### 6. Event Publishing
- Destination: CloudWatch, Data Firehose, hoặc SNS
- Track chi tiết: sends, deliveries, opens, clicks, bounces, complaints, rejects
- Có thể filter theo email characteristics
- Nền tảng để lưu `dispatch_events` vào Notification Service

---

## Kế hoạch áp dụng (Digima Notification Microservice)

| Phase | Mô tả | Cách dùng SES |
|-------|-------|--------------|
| **Phase 1** | Nhận bounce events → gửi Slack alert | SNS Feedback Notifications → webhook → Slack |
| **Phase 2** | Lưu dispatch_events, Admin App quản lý bounce | Event Publishing → SNS/Firehose → Notification Microservice |

> Phase 2 cần thiết kế tách biệt rõ ràng giữa `backend_app notifications` và `Notification Microservice`.

---

## Tài liệu tham khảo

- [AWS SES Monitor Sending Activity](https://docs.aws.amazon.com/ses/latest/dg/monitor-sending-activity.html)
