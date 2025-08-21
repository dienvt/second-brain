Query type

[https://confluence.zalopay.vn/x/p7rGBQ](https://confluence.zalopay.vn/x/p7rGBQ)

  

Cần deploy con bill tracking new CI/CD lên stg

- ActiveMQ mới hoàn toàn
- Off job (ok vì ko có trùng IP config chạy jobs)
- Dùng kafka khác hoàn toàn (không consumer kafka từ farm cũ). Con reminder sẽ chạy trên con mới (error)
- Không open cho core-api gọi vào (ok ko route traffic vào domain mới)

  

  

queryTypeEnum:

- Luồng mới sẽ cố gắn force dùng ServiceCode và providerID lấy từ phát query dẫn tới không cần phải lấy lại ở bước deliver.

  

  

Test SB

- [ ] NCB
- [ ] SHINHAN
- [ ] MCREDIT
- [ ] FECredit
- [ ]

  

  

# Rollout version

### App

STG

### ZPI

STG

### Provider Payoo

STG

### Core-api & config

PROD

Config đã thêm chưa enable

ERP prod chưa thêm

### Bill Tracking

Đã Deploy

### Notification

V2 sandbox

support version 7.14

  

32.48 →

# Check list

## Dev

---

- [ ] Kết nối và test thành công ở môi trường sandbox
- [ ] Cần có biên bảng nhiệm thu để bên đối tác chắc rằng ZaloPay đã test hết tất cả các case có thể xảy ra
- [ ] Tạo thông tin **App** cho từng môi trường QC, STG, PROD: Quannt6
- [ ] Thêm cấu hình **App (appID, hash_key, callback_key)** môi trường STG, PROD
- [ ] Cần đối tác cung cấp thông tin môi trường Production để tiến hành deploy lên môi trường staging/production

  

---

- [ ] Deploy provider & test provider ở môi trường STG
- [ ] Verifying the transactions recorded correctly (Zion + Partner)
- [ ] Check SFTP and Email
- [ ] Deploying to Production with 'inactive' mode

  

  

## OP,PA,BIZ

---

- [ ] Tạo các thông tin **Merchant Code**, **FA Code**: Business, OP
- [ ] Tạo thông tin **PaymentID** mapping với **FA Code**, **Merchant Code**

  

  

  

  

  

  

  

  

---

- [ ] Ký hợp đồng
- [ ] Chọn ngày lành tháng tốt go-live dịch vụ

  

  

  

# Other

- [ ] Logo của supplier đó đã có hay chưa
- [ ] Để nhắc nợ user cần gửi Notification reminder
- [ ] Cần data tracking ở FE để visualize lên dashboard
- [ ] Để có thể notify user về trạng thái của transaction cần Notification transaction dẫn tới LSGD v2
- [ ] Cần check notification nhắc nợ có đang mở được app hay không (in-app và out-app)

# Case study Payoo-CF

- [ ] Để có thể thanh toán phải active App 797, cần thông tin merchant code
- [ ] Để có thể query bill khi gọi sang Payoo cần phía Payoo mở cổng
    - [ ] SB cần biên bản nghiệm thu
    - [ ] Phải test các case phía Payoo yêu cầu
- [ ] Để có thể xem lại lịch sử giao dịch, phải show LSGD v2
- [ ] Để nhắc nợ user cần gửi Notification reminder
- [ ] Để có thể notify user về trạng thái của transaction cần Notification transaction dẫn tới LSGD v2
- [ ] Thay link ở trang more app (search app) đá về link revamp
- [ ] Cần data tracking ở FE để visualize lên dashboard
- [ ] Check SFTP and Email
- [ ] Cần confirm go-live do business gửi tới đối tác + các bên liên quan