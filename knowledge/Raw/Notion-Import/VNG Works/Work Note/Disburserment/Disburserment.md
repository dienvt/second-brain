- [ ] Update message
- [ ] Review update final status
- [ ] loadtest
- [ ] File Deploy : Dùng công nghệ gì?

  

# Deployment

sb -> 49.93.x

stg -> 109.3.x

prod -> 50.32.x

# TransType

APPID: 1228

transtype: 11

  

|Name|Assign|Status|
|---|---|---|
|[[ZMS]]|||
|[[STG Hot fix 1.0.8]]|||
|[[VNG Works/Work Note/Disburserment/Untitled Database/Card 3\|Card 3]]|||

  
  

  

[[docs]]

  

#### Domain

|Env|domain|IP: PORT|
|---|---|---|
|[[VNG Works/Work Note/Disburserment/Domain/SB\|SB]]|[sbgrpc-disbursement.zpapps.vn](http://sbgrpc-disbursement.zpapps.vn/)|10.109.3.86:8081|
|[[VNG Works/Work Note/Disburserment/Domain/STG\|STG]]|[stggrpc-disbursement.zpapps.vn](https://www.notion.sosbgrpc-disbursement.zpapps.vn)||
|[[VNG Works/Work Note/Disburserment/Domain/REAL\|REAL]]|||
|[[LOADTEST]]|||
|[[VNG Works/Work Note/Disburserment/Domain/Untitled\|Untitled]]|||

  
  

## Partner integration

1. Create new appID
2. Create and ZalopayID for partner ( MEP)

  

# Fundflow status

#### Fund Flow Status

|Name|Value|Description|
|---|---|---|
|[[FundFlowNotFound]]|0|Lỗi|
|[[FundFlowSuccess]]|1|Giao dịch thành công. Tiền đã trừ ví A và gửi ví B thành công.|
|[[FundFlowProcessing]]|3|Giao dịch đang xử lý|
|[[FundFlowTransitFailFundBackFail]]|7|Giao dịch bị pending. Tiền đã trừ ví A thành công nhưng cộng ví B thất bại, thực hiện auto fundback bị thất bại.|
|[[FundFlowTransitFailFundBackSuccess]]|6|Giao dich thất bại. Tiền đã trừ ví A thành công nhưng cộng ví B thất bại, thực hiện auto fundback thành công.|
|[[FundFlowTransitFailFundBackPending]]|8|Giao dịch bị pending. Tiền đã trừ ví A thành công nhưng cộng ví B thất bại, thực hiện auto fundback bị không biết rõ là đã fundback hay chưa.|
|[[FundFlowFail]]|2|Giao dịch thất bại. Trừ tiền ví A thất bại.|
|[[FundFlowPending]]|4|Giao dịch pending. Không rõ là đã trừ tiền ví A hay chưa|
|[[FundFlowTransitPending]]|5|Giao dịch pending. Trừ tiền ví A thành công nhưng không rõ là đã cộng tiền ví B hay chưa.|

  
  

[[Deployment]]

  

## Integrate new merchant

MEP prepare:

1/ tạo FACode, 2/ Map FACode với userId, zasID, 3/ Gắn FACode với merchant.

  

  

- Staging: {"user_id": "210312003500001", "accounting_code": "3013004001", "zas_id": "176069960307572758"}
- Production: {"user_id": "210625003500001", "accounting_code": "3013004001", "zas_id": "214149365215985686"}

  

```SQL
-- index orderID

-- 2021-07-12 10:48:56.6140
ALTER TABLE `stgdisbursement`.`orders` CHANGE `updated_at` `updated_at` datetime(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6) on update CURRENT_TIMESTAMP(6) COMMENT '';

 

-- 2021-07-12 10:49:03.3020
SELECT ordinal_position as ordinal_position,column_name as column_name,column_type AS data_type,character_set_name as character_set,collation_name as collation,is_nullable as is_nullable,column_default as column_default,extra as extra,column_name AS foreign_key,column_comment AS comment FROM information_schema.columns WHERE table_schema='stgdisbursement'AND table_name='orders';

 

-- 2021-07-12 10:49:04.7060
SELECT sub_part as index_length,index_name as index_name,index_type AS index_algorithm,CASE non_unique WHEN 0 THEN'TRUE'ELSE'FALSE'END AS is_unique,column_name as column_name FROM information_schema.statistics WHERE table_schema='stgdisbursement'AND table_name='orders'ORDER BY seq_in_index ASC;
```

![[/Untitled 6.png|Untitled 6.png]]

  

  

```Bash
{{disbursement-core local :8081 :8080 map[ae:lTvZ0nZX24GYt0SvnwzO mertool:7RMBr7vEmUTRmPA] 5} {10.109.3.21:4000 disbursement aDuCR8ubjm stgdisbursement} {10.109.3.50:6489  201506e01274381f4344e6bc2566415e51825a5be43ae3843d022222845c2 60} {true 10.109.3.86:6831} {stggrpc-zas.zalopay.vn:9501 Qz@xxZpN@93PnhF3 true 20011} http://10.50.32.3:3128 {stggrpc-disbursement.zalopay.vn:443 2 tWwecZE7B9x4MugeAzeAYrztCejhFnMh true 1228 38 MS001} {10.60.78.7:9095,10.60.78.8:9095,10.60.78.9:9095 ZP_CPS_TRANSFER_STATUS_STG mte_disbursement_core_stg -1} {stggrpc-umuserprofile.zalopay.vn:9075 true 24 tDGmBiY9zJ5wUc8Xi 2A472D4B61506453} {grpc-apimep-private.zpapps.vn:443 1 abc@123 false} {stggrpc-tpe.zalopay.vn:9905 16 Z5qnGYtDpA0ZnE4L true} {10.60.36.2:9095,10.60.36.3:9095,10.60.36.4:9095 ZPTransLogstg disbursement_core -1} {disbursement_core_mc } {10.50.49.45:9092,10.50.49.46:9092,10.50.49.47:9092 disbursement_core_handler_log_stg disbursement_core_order_log_stg disbursement_core} {map[ck:[162009399802986528]] @every 120s} {http://10.50.32.32:9092/httproxy/10.50.1.40:8989/limit/check-fund-in-limit 450 4 tester tester@qc} {true internal-socialstg.zalopay.vn:443 true https://scdn.zalopay.vn/zst/zpi/zms/transfer_no_confirm.png notice Nhận tiền thành công 1aed0c023047d9198056 oa.open.url https://socialstg.zalopay.vn/spa/?c=1 https://scdn.zalopay.vn/zst/zpi/zms/transfer_resend.png Xem chi tiết business template} ZaloPayClient}"}
```

  

# UM encrypt key

sb:"encryptUserIDKey":"y86jq2KAnq9rWnXv"

stg: "encryptUserIDKey":"PWyYXlGt7pdJD8Ua"

prod "encryptUserIDKey":"PWyYXlGt7pdJD8Ua"

[[VNG Works/Work Note/Disburserment/STG|STG]]