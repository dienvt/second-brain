  

Phase deploy lên trước rồi sau đó sẽ route traffic sau

# Checklist

- [ ] Xin DB admin
    
    - [x] STG:
    
    ```YAML
    ip: "10.109.3.21"
    port: 4000
    user: zpcps
    pass: "mep1123wasxcderc11root"
    dbName: "bill_reminder_stg"
    ```
    
    - [ ] REAL
    
    ```YAML
    ip: "10.109.3.21"
    port: 4000
    user: zpcps
    pass: "mep1123wasxcderc11root"
    dbName: "bill_reminder_stg"
    ```
    
- [x] Check bill-tracking `OUTSTANDING_BILL` Kafka brokers
    
    - [x] STG: `FileConfig.``_kafkaDataService_`
    
    ```YAML
    outstandingBillKafka:
    	brokers: "10.30.69.7:9095,10.30.69.8:9095,10.30.69.9:9095"
    	topics: "STG_OUTSTANDING_BILL"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```
    
    - [x] REAL `FileConfig.``_kafkaDataService_`
    
    ```YAML
    outstandingBillKafka:
    	brokers: "10.30.69.7:9095,10.30.69.8:9095,10.30.69.9:9095"
    	topics: "OUTSTANDING_BILL"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```
    
- [x] Lấy thông tin `CUSTOMER_METRICS` push data to ZDS
    
    - [x] STG:
    
    ```YAML
    updatedBillKafka:
    	brokers: "10.30.69.7:9095,10.30.69.8:9095,10.30.69.9:9095"
    	topics: "STG_CUSTOMER_METRICS"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```
    
    - [x] REAL
    
    ```YAML
    updatedBillKafka:
    	brokers: "10.30.69.7:9095,10.30.69.8:9095,10.30.69.9:9095"
    	topics: "CUSTOMER_METRICS"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```
    
- [x] Notify & ZMS Kafka
    
    - [x] STG:
    
    ```YAML
    zmsKafka:
      brokers: "{{aqr.bill.remind.zms-kafka.brokers}}"
      topics: "STG_CPS_NOTIFYMANAGEMENT_ZMS"
    
    notifyKafka:
      brokers: "{{aqr.bill.remind.notify-kafka.brokers}}"
      topics: "STG_CPS_NOTIFYMANAGEMENT_NOTIFY"
    ```
    
    - [x] REAL
    
    ```YAML
    zmsKafka:
      brokers: "{{aqr.bill.remind.zms-kafka.brokers}}"
      topics: "CPS_NOTIFYMANAGEMENT_ZMS"
    
    notifyKafka:
      brokers: "{{aqr.bill.remind.notify-kafka.brokers}}"
      topics: "CPS_NOTIFYMANAGEMENT_NOTIFY"
    ```
    
- [x] Relation
    
    - [x] STG:
    
    ```YAML
    updatedBillKafka:
    	brokers: "{{aqr.bill.remind.outstanding-bill-kafka.brokers}}"
    	topics: "STG_CUSTOMER_METRICS"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```
    
    - [x] REAL
    
    ```YAML
    updatedBillKafka:
    	brokers: "{{aqr.bill.remind.outstanding-bill-kafka.brokers}}"
    	topics: "QC_CUSTOMER_METRICS"
    	group_id: "bill_reminder_group"
    	offset: -1
    ```