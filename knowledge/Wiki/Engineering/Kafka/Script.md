# Kafka script
## start broker
```bash
cd kafka_2.11-2.0.0
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```
## create topic
```bash
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 12 --topic [topicname]
```
## list topic
```bash
bin/kafka-topics.sh --list --zookeeper localhost:2181
```
## see info
```bash
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic testkafka
```
## send message
```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic [topic]
```
## consume message
```bash
bin/kafka-console-consumer.sh --topic [topic] --from-beginning --bootstrap-server localhost:9092

# with group name
bin/kafka-console-consumer.sh --topic BillTrackingCrawler --bootstrap-server 10.50.49.2:9092,10.50.49.3:9092,10.50.49.4:9092  -from-beginning --consumer-property group.id=CPS_CORE_API 

```
## Update partitions

### Describer
```bash
/bin/kafka-log-dirs.sh --describe --bootstrap-server localhost:9092 --topic-list [topic]
```

### Alter partition size
```bash
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --topic CPS_promotion_order_logs  --partitions 12 
```

## kafka lag
```bash
bin/kafka-consumer-groups.sh --bootstrap-server broker1:9092 --describe --group [group name]
```

bin/kafka-topics.sh --list --zookeeper 10.30.83.3:2181

bin/kafka-console-consumer.sh --topic disbursement_core_handler_log --from-beginning --bootstrap-server 10.109.3.47:9092

bin/kafka-console-consumer.sh --topic CPS_COMMON_ORDER --from-beginning --bootstrap-server 10.60.45.6:9092

--- very old version 
/zserver/kafka/kafka_2.11-0.9.0.0/bin/kafka-console-consumer.sh --topic CPS_COMMON_ORDER --from-beginning --bootstrap-server 10.60.45.2:9092,10.60.45.5:9092,10.60.45.6:9092 --new-consumer


## Console Consume
kafka-console-consumer.sh --bootstrap-server 10.109.3.47:9092,10.109.3.48:9092,10.109.3.49:9092 --topic CPSOrderLog --property print.key=true --property key.separator="-" --from-beginning

```bash
kafka-console-consumer.sh --bootstrap-server 10.60.45.2:9092,10.60.45.5:9092,10.60.45.6:9092 --topic CPS_COMMON_ORDER --property print.key=true --property key.separator="-" --from-beginning
```
kafka-console-producer.sh --broker-list 10.109.3.47:9092,10.109.3.48:9092,10.109.3.49:9092 --topic MC_CPSOrderLog

bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092


  kafka-console-consumer.sh --bootstrap-server 10.109.3.47:9092,10.109.3.48:9092,10.109.3.49:9092 --topic DEV_BILL_STORAGE --property print.key=true --property key.separator="-" --from-beginning

kafka-console-producer.sh --broker-list 10.109.3.52:9092,10.109.3.53:9092,10.109.3.54:9092 --topic CPSOrderLog

kafka-console-consumer.sh --bootstrap-server 10.60.45.2:9092,10.60.45.5:9092,10.60.45.6:9092 --topic QC_CPS_NOTIFYMANAGEMENT_NOTIFY --property print.key=true --property key.separator="-" --from-beginning --consumer-property group.id=test1

# Offset shipper
reference: [https://stackoverflow.com/a/49542334](https://stackoverflow.com/a/49542334 "https://stackoverflow.com/a/49542334")

```shell
kafka-consumer-groups.sh --bootstrap-server loclahost:9092 --group xxx-0 --topic schedule-changed --reset-offsets --shift-by -2 --execute
```

```shel
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupname --describe
```

ship by date 
```shell
kafka-consumer-groups.sh --bootstrap-server 10.50.49.2:9092,10.50.49.3:9092,10.50.49.4:9092 --reset-offsets --group CPS_CORE_API --topic BillTrackingCrawler --to-datetime 2022-09-07T01:00:00.000+07:00 --dry-run

kafka-consumer-groups.sh --bootstrap-server 10.50.49.2:9092,10.50.49.3:9092,10.50.49.4:9092 --reset-offsets --group CPS_CORE_API --topic BillTrackingCrawler --to-datetime 2022-09-07T01:00:00.000+07:00 --execute

```



```
bin/kafka-consumer-groups.sh --bootstrap-server 10.50.49.45:9092,10.50.49.46:9092,10.50.49.47:9092 --describe --group prod.QUERY_HUB_INQUIRY_REGULATOR.PAYOO
```

./kafka-consumer-groups.sh --bootstrap-server 10.50.49.45:9092,10.50.49.46:9092,10.50.49.47:9092 --describe --group prod.QUERY_HUB_INQUIRY_REGULATOR.PAYOO