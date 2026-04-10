# Cross sale approach:

1. Sau khi gọi **Cashier Page** để thanh toán bill điện, app sẽ hiện **Result Page**
    1. Sử dụng **customize area** để chèn deep link đá vào **Bill Detail Page** với produce insurance
        1. Cách này sẽ giật lag vì phải đá sang home trước khi đá vào detail
        2. ZPI sẽ mượt hơn ZPA
    2. Sau khi user click nút **Đóng** sẽ hiện **Bill Detail Page** với produce insurance
        
        1. FE phải handle event đóng app
        
          
        
2. Vấn đề về SSO của Saladin

# Auto-debit approach:

1. Cashier page không thể hiện số tiền tổng của 2 giao dịch mà chỉ hiện được của GD gốc
    1. nhắc PO cần cân nhắc UX chỗ này ko rất confuse với user
    2. Có thể add thêm thông tin cho khách hàng ở phần customize result page
2. Khi tiến hành binding cần PIN của user (đã hỏi anh HA). Luồng auto debit chạy sau luồng thanh toán nên cũng có thể là user pay bill điện 1 lát sau > 1m thì mới thông báo bị trừ tiền do auto-debit (UX tệ)
    1. Luồng này có thể tuỳ thuộc luồng deliver insur nằm ở đâu
3. Momo đang một màng hình thanh toán cho 2 orders (anh Hoàng Anh confirm)

# Saladin API

### Query bill fee

Input: user phone, bill outstanding amount

output: fee amount

### Buy

Input: Need user KYC info

output: Contrast identity

  

Không bỏ được API 1

single sign-on