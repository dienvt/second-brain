Config card table

VNMB 10k → 200 80

  

# fix security

lên stg zp-cs-service

[https://itsm.zalopay.vn/change/edit/C25067?sign=923c18962efb5d90cf9dafc9e43f0b6fb7e4b7e4](https://itsm.zalopay.vn/change/edit/C25067?sign=923c18962efb5d90cf9dafc9e43f0b6fb7e4b7e4)

  

lên pro zp-cs-exportserivce

  

lên pro zp-cs-service

  

Anh thấy postpaid core đã từ supplierID sang ProviderID và providerCode rồi thì chỗ này có cần phải mapping lại không?

  

Anh thấy function này có chút vấn đề là nếu một session expired thì sẽ có nhiều request renew session. Chỗ này có cần check thử session đã được renew chưa không?