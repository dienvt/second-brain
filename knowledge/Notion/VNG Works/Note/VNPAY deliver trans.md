# Root cause:

Kết nối sang VNPay chập chờn dẫn tới service tự động restart.

Không có graceful shutdown nên thường những request sẽ bị ngắt giữa chừng.

Những lúc này CS cần DEV confirm giao dịch refund (CS cung cấp list orderID)

# How to resolve?

VNPay provider có cache lại transID tương ứng với orderID ngay khi tạo ra. Vào cstool vào mục pending order ([https://cps-cstool.zpapps.vn/billing/operation/pending/get](https://cps-cstool.zpapps.vn/billing/operation/pending/get)) để gọi get status là sẽ có transID.

  

Trong trường hợp nhiều transaction pending và không sure (call get status bị lỗi connect redis cũng trả về -400 cho nên -400 ko tin cậy dẫn tới phải check nhiều lần)

  

Lúc này chạy scrip sau để kiểm tra redis có đang chứa value hay không

  

```Shell
#!/bin/sh
PREFIX="PROVIDER-VNPAY-REAL:PROVIDER-TRANS:"
redisKey(){
    K=$PREFIX$1
    echo GET $K | redis-cli --no-auth-warning -h <host> -p <port> -a <pass>
}
while read line; do
    ret=$(redisKey $line)
    echo "$line:$ret"
done < data.txt
```

data.txt sample:

```Java
22070200006495
22070200006468
22070200006518
22070200006504
22070200006454
22070200006487
```

kết quả sẽ in ra console theo dạng:

```Java
22070400008917:50201�
22070400008918:50201�
22070400008893:50201�
22070400008942:50201�
22070400008912:
22070400009023:
```

Sau đó chỉ cần check từng dòng order có value ở tool pending order ([https://cps-cstool.zpapps.vn/billing/operation/pending/get](https://cps-cstool.zpapps.vn/billing/operation/pending/get))