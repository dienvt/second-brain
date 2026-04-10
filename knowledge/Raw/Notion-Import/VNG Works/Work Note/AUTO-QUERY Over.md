# Measure

→ tách v1, v2, self?

áp dụng luôn tách queue?

  

  

# Query bill before notification

query có updateDate trong vòng 7 ngày chạy lúc 2h sáng

# Optimize query bill

95% vs 5%

  

  

PB05020021561 --> có 2nd và 3rd noti vào ngày 14 và 19/7, ko có 1st noti

- splunk: có init 3 phát noti
    - [Init-RemindBill] msg={"appid":17,"supplierid":5,"customercode":"PB05020021561","noNoti":1,"billstatus":0,"cycletype":1,"cyclevalue":"202207","billtype":1,"amount":1048104,"zalopayID":"","customername":"Đỗ Văn Sên","customeraddress":" Bến Rộng, Thạnh Đức, GD, TN. T68/116/9","notiDate":0,"notiTime":0,"dueTime":0}, delay=13619,
    - [Init-RemindBill] msg={"appid":17,"supplierid":5,"customercode":"PB05020021561","noNoti":2,"billstatus":0,"cycletype":1,"cyclevalue":"202207","billtype":1,"amount":1048104,"zalopayID":"","customername":"Đỗ Văn Sên","customeraddress":" Bến Rộng, Thạnh Đức, GD, TN. T68/116/9","notiDate":0,"notiTime":0,"dueTime":0}, delay=186395,
    - [Init-RemindBill] msg={"appid":17,"supplierid":5,"customercode":"PB05020021561","noNoti":3,"billstatus":0,"cycletype":1,"cyclevalue":"202207","billtype":1,"amount":1048104,"zalopayID":"","customername":"Đỗ Văn Sên","customeraddress":" Bến Rộng, Thạnh Đức, GD, TN. T68/116/9","notiDate":0,"notiTime":0,"dueTime":0}, delay=618457,
- splunk: chỉ có push noti lần 2:
- data2: ko có data

  

  
PD20007524511 --> có 3rd noti vào 19/7, ko có 1st và 2nd noti