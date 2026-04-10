- Nhu cầu:
    - Insert
    - Query
        - theo appid, orderID, zalopayID → single
        - theo appid, zpTransID, zalopayID → single
        - theo appID, zalopayID → list
        - theo appID, zalopayID + timestamp → list
    - Update
        - refund ?? cần ko
        

Vậy ra chỉ cần mỗi cái history sync chứ mấy ???

Todo

- [x] check toll history format
    
    ```JavaScript
    {
        "appID": 599,
        "amount": 200000,
        "zaloPayID": "190215000002593",
        "zpTransID": "231105001411527",
        "providerTransID": "51091405",
        "providerResultCode": "1",
        "refundStatus": 0,
        "refundID": "",
        "orderStatus": 1,
        "providerCode": "VETC",
        "supplierName": "VETC",
        "userFeeAmount": 3400,
        "discountAmount": 5000,
        "orderTime": 1699198035000,
        "embedData": "{\"columninfo\":\"{\\\"customercode\\\":\\\"E0100601627\\\",\\\"data\\\":{\\\"bills\\\":[{\\\"amount\\\":200000,\\\"fee\\\":0}]}}\",\"zlppaymentid\":\"\"}",
        "item": "[]",
        "description": "Nạp tiền VETC",
        "pmcID": 38,
        "bankCode": "",
        "orderID": 2311051196486475,
        "mAppUser": "190215000002593",
        "supplierID": 550,
        "appInfo": "",
        "hisAppInfo": "{\"billid\":\"2311051196486475\",\"customercode\":\"E0100601627\",\"customername\":\"\",\"customeraddress\":\"\",\"providertransid\":\"51091405\",\"monthstr\":\"11/2023\",\"month\":202311}"
    }
    ```
    
- [ ] Implement consume migrate
- [ ] Design data model
    - [ ] Đang dùng appID để sharding
    - [ ] Logic show by desc orderTime
    - [ ] hisappinfo{  
        orderstatus  
        month  
        customercode  
        month  
        description  
        ordertime  
        amount  
        }  
        
- [ ] Check DataX push data to Kafka
- [ ] Consume kafka to write to DB
- [ ] Deploy