# Overview

#### Module

|Name|Git-repo|Service|Note|
|---|---|---|---|
|[[EVNGATEWAY1]]|[https://gitlab.zalopay.vn/billing/providers/zp-cps-provider-evngateway.git](https://gitlab.zalopay.vn/billing/providers/zp-cps-provider-evngateway.git)|EVN-HCM, EVN-HN, EVN-SPC|EVN SPC chỉ đi Direct không qua PAYOO|
|[[EVNGATEWAY2]]|[https://gitlab.zalopay.vn/billing/providers/zp-cps-provider-evngateway.git](https://gitlab.zalopay.vn/billing/providers/zp-cps-provider-evngateway.git)|EVN-CPS|Pro using branch support_v2|
|[[EVNGATEWAY3]]|[https://gitlab.zalopay.vn/billing/providers/evn.git](https://gitlab.zalopay.vn/billing/providers/evn.git)|EVN-NPC|Mục tiêu là move hết 2 cái kia qua cái mới này.  <br>Code anh Vĩnh đã làm sẵn hết r chỉ có spc là chưa có source.  <br>  <br>CPC không có update v2 nên hỏi biz khỏi làm nghiệm thu, test kỹ rồi switch qua luôn cho khoẻ bỏ được cái branch support_V2|

  
  

# Detail

## Northern

[[EVN HN]]

[[EVN NPC]]

## Middle

[[EVN CPC]]

## Southern

[[EVNSPC]]

[[EVN HCM]]

# Script

```SQL
USE zpCPSPlatformAdmin;

# table AppSupplierProvider
INSERT INTO AppSupplierProvider (appID, supplierID, providerCode, status, providerAppServiceID, providerSupplierCode, weight, `order`) VALUES (429, 5, 'EVN_SPC', '1', 'DIEN', 'EVNSPC', 1, 1);

# table ProviderApiConfig
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'baseUrl', 'https://zlpdev-bill-provider-evnspc.zalopay.vn', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'clientID', '1', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'hashKey', 'abc123', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'deliverPath', '/v1/trans/deliver', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'redeliverPath', '/v1/trans/redeliver', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'getStatusDeliverPath', '/v1/trans/status', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'queryBillPath', '/v1/bill/info', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'maxRetryGetStatus', '5', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'providerCode', 'EVN_SPC', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'providerName', 'Điện Miền Nam', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'secondDelayGetStatus', '3', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'secondTimeout', '120', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'updateStatusDeliverPath', '/v1/trans/update', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'updateStatusProvider', '1', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'useProxy', '1', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'useGrpc', '0', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'grpcPoolSize', '4', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'grpcSecondTimeout', '0', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'grpcServerAddress', '10.205.21.19:6981', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'grpcStatus', '0', '');
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'grpcUseSSL', '0', '');


// chieu làm
USE zpCPSPlatformAdmin;
INSERT INTO ProviderApiConfig (section, `key`, value, description) VALUES ('EVN_SPC', 'redeliverPath', '/v1/trans/redeliver', '');


PA09055021501

https://jk.zpapps.vn/blue/organizations/jenkins/billing-evn-providers/detail/billing-evn-providers/201/pipeline

Merchant_DataCentralize_Billing
Merchant_DataCentralize_GameCard
Merchant_DataCentralize_Telco

Merchant_DataCentralize_Billing_STG
Merchant_DataCentralize_GameCard_STG
Merchant_DataCentralize_Telco_STG

CPS_CORE_API_MONITOR_LOG
CPS_CORE_API_MONITOR_LOG_STG
CPS_CORE_HANDLER_LOG
CPS_CORE_HANDLER_LOG_STG
CPS_CORE_ORDER_LOG
CPS_CORE_ORDER_LOG_STG
```