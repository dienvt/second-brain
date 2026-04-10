Hi mn,

Để tối ưu hoá chất lượng sản phẩm thì có 2 point Điền muốn raise lên:

1. **Monitor owner:**
    1. Việc monitor các service của service bill trước giờ không rõ ràng. Điền đánh giá cao tính owner ship của mọi người. Nhưng mà vẫn phải làm rõ chuyện này.
    2. Ngoài giờ làm: Điền muốn để việc monitor sang 247, tuy nhiên để 247 có thể monitor hiệu quả thì cần mọi người correct các dashboard mà mình owner và note description từng board.
2. **Log severity**: Sắp tới anh Vũ có apply rule alert đối với log level để giảm thời gian phát hiện và xử lý sự cố. Để mọi người thực sự aware và tuân thủ thì sẽ **có chế tài nếu vi phạm** và đương nhiên sẽ có **phần thưởng nếu thực hiện tốt**. Sau đây là các level anh Vũ đã define và Điền có note thêm một số concern:
    1. FATAL
        1. Khi init lỗi hệ thống crash, lỗi system
        2. cần alert & action
        3. Technical issue
    2. ERROR
        1. Connect up/down stream ko được: Gọi third parties ko được (network hay là errors)?
        2. Reload config lỗi (Warning thôi chứ sao lại là error tại vì reload fail thì đâu có được afffect)
        3. Technical issue
    3. WARNING
        1. có behavior bất thường về mặt biz: user login sai mk, amount quá lớn..
        2. check rule fail bất thường
        3. Business Issues & Technical Issue
    4. INFO
        1. Bình thường để log thông tin
        2. Một số service đang in quá nhiều log nên cần review và xoá đi
        3. Business Issues & Technical Issue
    5. DEBUG
        1. Debug log ở môi trường sandbox

  

Hiện tại những điểm trên đang trong quá trình define nên nếu có điều gì bất cập mong nọi người phản hồi sớm. Nếu ko có phản hồi thì sau ngày 20/10/2023 thì mặc định mọi người chấp nhận các điều khoản trên.