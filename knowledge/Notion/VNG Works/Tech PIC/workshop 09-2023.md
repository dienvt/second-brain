Toàn bộ tài liệu: [https://vngms.sharepoint.com/sites/ZaloPayTechPICCommunity/Shared Documents/Forms/AllItems.aspx?csf=1&web=1&e=RxWyia&cid=5adbf374-113e-4505-90b4-13a02eee145e&RootFolder=%2Fsites%2FZaloPayTechPICCommunity%2FShared Documents%2FGeneral%2F2023 Workshop%2FSep 23 Workshop&FolderCTID=0x01200095AD31D2CA394448A320838B65B23954](https://vngms.sharepoint.com/sites/ZaloPayTechPICCommunity/Shared%20Documents/Forms/AllItems.aspx?csf=1&web=1&e=RxWyia&cid=5adbf374%2D113e%2D4505%2D90b4%2D13a02eee145e&RootFolder=%2Fsites%2FZaloPayTechPICCommunity%2FShared%20Documents%2FGeneral%2F2023%20Workshop%2FSep%2023%20Workshop&FolderCTID=0x01200095AD31D2CA394448A320838B65B23954)

# Session 1

Phúc present về cách rate limit. Thì đang dùng envoy để implement, cách rate sẽ là config một key nào đó trong request header. Ví dụ request header có key zaloPayID thì sẽ set rate limit trên key đó. Trong slide có mô tả kiến trúc sẽ như thế nào.

- Phase tới sẽ có tool để tech PIC có thể vào tự config số lượng rate và key dùng để rate limit
- Anh Bằng có hỏi về latency khi round trip trên 2 con gateway ( 1 con là nginx từ đầu ngoài vào và 1 con là envoy) thì anh Hưng có bảo thằng Envoy có cơ chế circuit breaker nên lỡ bị gì thì nó sẽ by pass.
- Cả phòng tranh cãi về việc dùng key nào vì header key có thể đổi được. Cuối cùng không kết luận được gì

# Session 2

Quang present về PII. Phần này nêu mấy case study bị leak PII với thêm một số kĩ thuật để bảo vệ thông tin PII. Slide có nêu rõ mấy điểm đó.

# Session 3

Anh Hưng với anh Huy present về fund security. Anh Hưng nhấn mạnh là e-wallet dễ bị tấn công ( số liệu có trong slide).

- Tương lai thì khi làm feature nào dính tới fund movement thì sẽ có một cái checklist về fund security để mọi người làm theo
- Sau đó anh Huy có present về các kiểu fund security và case study ở ZaloPay. Về nội dung các kiểu thì có trong slide. Case study thì có:
    - vụ không consist status khi gọi client ( server thì trả về process status còn client thì hiểu là order status nên có trường hợp process success nhưng order fail)
    - Các hệ thống không consistent vì thiếu hoặc implement sai cơ chế lock
    - Về hệ thống ZAS có test kĩ và có cả chaos testing
    - Case số tiền bank charge khác với số tiền order
    - Các state của order ko rõ ràng vì ngày xư đi theo error code
    - Fund loss trong trường hợp timeliness là user đứng ở quầy disbursment cần thời gian chốt status nhanh