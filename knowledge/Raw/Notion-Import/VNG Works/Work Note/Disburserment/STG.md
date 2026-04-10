domain: "stggrpc-disbursement.zpapps.vn"

useTLS: true

  

```JSON
{
    "app":
    {
        "name": "disbursement-core",
        "mode": "stg",
        "http_addr": ":8080",
        "grpc_addr": ":8081",
        "client_map":
        {
            "ae": "lTvZ0nZX24GYt0SvnwzO",
            "mertool": "7RMBr7vEmUTRmPA"
        },
        "max_query_minute": 5
    },
    "mysql":
    {
        "addr": "10.109.3.21:4000",
        "user": "disbursement",
        "pass": "aDuCR8ubjm",
        "db_name": "stgdisbursement"
    },
    "redis":
    {
        "addr": "10.109.3.50:6489",
        "pass": "201506e01274381f4344e6bc2566415e51825a5be43ae3843d022222845c2",
        "expiration": 60
    },
    "tracing":
    {
        "enable": true,
        "addr": "10.109.3.86:6831"
    },
    "mte":
    {
        "addr": "stggrpc-disbursement.zalopay.vn:443",
        "client_id": "2",
        "client_key": "tWwecZE7B9x4MugeAzeAYrztCejhFnMh",
        "use_tls": true,
        "app_id": 1228,
        "pmc_id": "38",
        "product_code": "MS001"
    },
    "mte_kafka":
    {
        "brokers": "10.60.78.7:9095,10.60.78.8:9095,10.60.78.9:9095",
        "topics": "ZP_CPS_TRANSFER_STATUS_STG",
        "group_id": "mte_disbursement_core_stg",
        "offset": -1
    },
    "zas":
    {
        "addr": "stggrpc-zas.zalopay.vn:9501",
        "token": "Qz@xxZpN@93PnhF3",
        "use_tls": true,
        "system_id": 20011
    },
    "um":
    {
        "addr": "stggrpc-umuserprofile.zalopay.vn:9075",
        "client_id": "24",
        "hash_key": "tDGmBiY9zJ5wUc8Xi",
        "use_tls": true,
        "m_private_key": "PWyYXlGt7pdJD8Ua"
    },
    "tpe":
    {
        "addr": "stggrpc-pe-adapter.zalopay.vn:443",
        "client_id": "16",
        "key": "Z5qnGYtDpA0ZnE4L",
        "use_tls": true
    },
    "tpe_kafka":
    {
        "brokers": "10.60.36.2:9095,10.60.36.3:9095,10.60.36.4:9095",
        "topics": "ZPTransLogstg",
        "group_id": "disbursement_core",
        "offset": -1
    },
    "mep":
    {
        "addr": "grpc-apimep-private.zpapps.vn:443",
        "client_id": "1",
        "key": "abc@123",
        "use_tls": false
    },
    "alert_balance":
    {
        "partner_zid":
        {
            "CK": [162009399802986528]
        },
        "spec_alert": "@every 120s"
    },
    "metrics":
    {
        "namespace": "disbursement_core_stg",
        "subsystem": ""
    },
    "kafka":
    {
        "brokers": "10.50.49.45:9092,10.50.49.46:9092,10.50.49.47:9092",
        "handle_log_topics": "disbursement_core_handler_log_stg",
        "order_log_topics": "disbursement_core_order_log_stg",
        "group_id": "disbursement_core"
    },
    "um_limit":
    {
        "addr": "http://10.50.32.32:9092/httproxy/10.50.1.40:8989/limit/check-fund-in-limit",
        "app_id": 450,
        "trans_type": 4,
        "client_id": "tester",
        "api_key": "tester@qc"
    },
    "zms":
    {
        "enable": true,
        "addr": "internal-socialstg.zalopay.vn:443",
        "use_tls": true,
        "banner": "https://scdn.zalopay.vn/zst/zpi/zms/transfer_no_confirm.png",
        "notice": "Có thông báo",
        "template_data_title": "Nhận tiền thành công",
        "template_id": "1aed0c023047d9198056",
        "action_type": "oa.open.url",
        "action_url": "https://socialstg.zalopay.vn/spa/?c=1",
        "image_url": "https://scdn.zalopay.vn/zst/zpi/zms/transfer_resend.png",
        "footer_title": "Xem chi tiết",
        "template_type": "business",
        "zms_type": "template"
    },
    "qr_user_agent": "ZaloPayClient",
    "http_proxy": "http://10.50.32.3:3128"
}
```