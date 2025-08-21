# Technical note

# Process note

[[ZMS & Notification]]

[[ActiveMQ]]

[[HMAC không chính xác]]

- Product code (Cashier Page)
    
    productcode liên quan vụ cashier createorder thì luôn là AC002 với các giao dịch bill telco.
    
    Truyền AC002  
    Default: AC005  
    

[[VNPAY deliver trans]]

[[VNPT]]

[[PAYOO]]

[[Contact Point]]

[[Planning Agenda]]

[[interview agenda]]

[[Squad meeting agenda]]

[[IDP]]

---

  

# Issue List

  

# Sarama consume

```Go

select {
case message := <-claim.Messages():
   if consumer.cb(message.Topic, message.Value) {
      session.MarkMessage(message, "")
   }
case <-session.Context().Done():
   return nil
}

return nil
```

  

# Auto debit

Là op tôi cần view lại những giao dịch auto debit đang bị pending? chưa paid trong 1 tháng? và có thể action pay.

  

# Agreement pay

v1: đi sync

v2: đi async

Xử lý notify cho hoá đơn agreement ở phía core-api. send kafka?

core-api chưa có luồn tích hợp qua luồng mới: phải tích hợp thêm.

  

  

## SSE vs Web socket

Socket đang đi qua nhiều proxy dẫn tới không ổn định

Applications ↔ NginX(K8s) ↔ Gateway ↔ Service

  

Giải pháp:

Dùng SSE thay thế

### SSE

TCP protocol

Mono-directional (only allowing the client to receive data from the server).

Transmit only UTF-8 data

SSEs suffer from a limitation to the maximum number of open connections, which can be especially painful when opening various tabs, as the limit is per browser is six.

### WebSocket

TCP protocol

Bi-directional (allowing communication between the client and the server)

Transmit both binary and UTF-8 data

  

TCP

## HMAC Billing

Fix ở core API thay thế unicode char bằng valid char

  

# Supplier

  

# Notification

old format: zalopay://…..

new format: https://……

Old format can’t open new UI (that resonable)

New format can’t open old UI (that not aceptable)

  

check data?

# Book từng người phỏng vấn:

## Câu hỏi:

1. Vai tròng trong team là gì? đang làm việc với ai nhiều nhất ( trong team lẫn ngoài team)? Có gặp khó khăn gì trong lúc làm việc hay không?
2. Ở góc nhìn cá nhân thì hệ thống bill đang như thế nào?

  

  

  

# CF

Landing page lúc switch sang cashier.

Cần DS bệnh viện → link

Tình hình CF hiện tại:

ZPI: Vẫn ổn

ZPA:

- notify: đang nhờ ThanhNV7 support config cho app 797, này cần chị Hà. Trần Thùy Nhi (2) nói chuyện với PO bên đó, case y chang app 17, 18 hồi trước (Nam said)
- mở app:
    
    - Android: chưa mở được, chuẩn bị nhờ CuongBV2 support debug
    - ==iOS: mở được, từ app Bill qua CF -> OK, CF qua Bill -> cái deeplink đang bị lỗi, đang redirect param tùm lum -> Chưa có hướng xử lý tiếp theo==
    
      
    

Off v1:

1. OFF Query V1 - chỉ chạy luồng V2
2. A Vũ + Điền: Nghĩ câu lệnh add thêm vào V2 chạy bù nếu số lượng reminder sụt giảm sau khi off V1
3. Câu query v2 ( bill đã có hoá đơn cách hôm nay 2 ngày/7 ngày , due date)
4. Diễm làm việc với Linh - Châu lấy data list bill dc nhắc nợ ngày 8/8 (bi tràn) - so với list bill nhắc nợ 8/7
5. Sáng mai 10/8: Xem lượng nhắc nợ có bị biến động ntn so với 10/7