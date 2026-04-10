# Overview

Start date: **July 30, 2018**

  

Diagram [https://drive.google.com/file/d/1dCbHW1Y5zN0v1wS_RBcSZ9eAiHMpJHFv/view?usp=sharing](https://drive.google.com/file/d/1dCbHW1Y5zN0v1wS_RBcSZ9eAiHMpJHFv/view?usp=sharing)

Jira [https://jira.zalopay.vn/secure/RapidBoard.jspa?rapidView=231](https://jira.zalopay.vn/secure/RapidBoard.jspa?rapidView=231)

**DDOS dns not able to resolve dns so callback can't reach core service**

# Mapping FA code

group team: **Mapping FA Code & Create PaymentIds**

# 🚂 Road to V2 💯

## Issue

- High coupling
    
      
    
- Logging topic
- Monitoring

  

## Roadmap

- Loose coupling
    
    Move to domain layer
    

# Reconcile

[[Bill settle]]

  

# History

Detail thì từ lsgd -> đúng , còn result page thì từ con SSE của zpi backend bắn đi

  

#### Task

|Name|Assign|Status|
|---|---|---|
|[[ZDS handler log]]||In progress|
|[[Send userID when query bill]]|DDien Vo|In progress|
|[[New CI-CD]]|||
|[[Bảo trì Billtracking và bài toán billing]]|||
|[[Core metric enhance]]|||
|[[centralize billing docs]]|||
|[[Security billtracking]]|DDien Vo||
|[[Update returnCode]]|DDien Vo||
|[[Payoo new supplier]]|DDien Vo|Not started|
|[[Entity type complicate between ZPI and ZPA]]|||
|[[BillTracking docs]]|DDien Vo|Completed|
|[[Deploy bill-core pro]]||Completed|
|[[05.2021 CycleValue = 0]]||Completed|
|[[Release core 3.5.0]]|DDien Vo|Completed|
|[[Group All 5 EVN]]|DDien Vo|Completed|
|[[Deploying 3.5.0]]|DDien Vo|Completed|

  
  

  

#### Operation Note

|Name|Assign|Status|
|---|---|---|
|[[Maintain]]|||
|[[Error Code update]]|||
|[[VNG Works/Work Note/Billing/Operation Note/Card 3\|Card 3]]|||

  
  

  

#   
Supper set  

```Plain
-- so luong ma kh input moi
select count(DISTINCT customercode) from insert_customer_log as icl where ym = '202101' and not exists(
    select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
)


select * from insert_customer_log where customercode ='PE10000013303'
-- duoc remind
select count(DISTINCT customercode) from remind_bill_log where ym = '202102' and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
)

-- bi xoa
select count(DISTINCT customercode) from unlink_customer_log where (ym = '202102' or ym = '202101')  and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
)

-- duoc auto query
SELECT count(DISTINCT customercode) from auto_query_log where ym = '202102' and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
)

-- duoc update ra bill da thanh toan
SELECT count(DISTINCT customercode) from handle_update_bill_log where ym = '202102' and billstatus=2 and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
)

select DISTINCT customercode from insert_customer_log as icl where ym = '202102' and ymd ='2021-02-15' and not exists(
    select * from insert_customer_log as iclo where ym != '202102' and iclo.customercode=icl.customercode
) and not exists(
    SELECT * from auto_query_log aql where ym = '202103' and aql.customercode=icl.customercode
)

select * from insert_customer_log where customercode='PE06000105883'

-- remove absolutly
select count(DISTINCT customercode) from remove_customer_log  where ym = '202102'


SELECT count(DISTINCT customercode) FROM handle_update_bill_log hubl where zalopayid!='' and ym='202102' and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
) and not exists(
    SELECT * FROM handle_update_bill_log bublo where zalopayid ='' and bublo.customercode=hubl.customercode
)

SELECT count(DISTINCT customercode) FROM handle_update_bill_log hubl where zalopayid!='' and ym='202101' and customercode in (
    select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
        select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
    )
)

select DISTINCT customercode from insert_customer_log as icl where ym = '202101' and not exists(
    select * from insert_customer_log as iclo where ym != '202101' and iclo.customercode=icl.customercode
) and not exists (
    SELECT * FROM handle_update_bill_log as hubl where zalopayid!='' and ym='202101' and hubl.customercode=icl.customercode
)


SELECT * FROM insert_customer_log where customercode='PP09000918864'

SELECT * FROM handle_update_bill_log as hubl where customercode='PB08050109059'

SELECT * FROM evn_zns_follow_log
SELECT * FROM evn_zns_msg_log where ntf_customer_code='PP09000918864'
SELECT * FROM evn_zns_after_zns_log where source like 'PP09000918864%'

```

  

[[Task Tracking]]