|Name|Assign|Status|
|---|---|---|
|[[ViettelPay Inter Skeleton]]|||
|[[ViettelPay Inter Server]]|||
|[[ViettelPay Inter Handler]]|||
|[[Card 3]]|||

  
  

# Core API

```Plain

[app]
....
dbLogBackupThresholdDate=210801

[db-topup-log-bk]
driver=com.mysql.jdbc.Driver
user=dbgtest
password=abc@123
url=....
poolSize=1
cachePrepStmts=true
prepStmtCacheSize=250
prepStmtCacheSqlLimit=2048
enableExternalPoolLib=true
```

  

# Admin Tool

## STG and REAL

```Plain
[db-topup-log-bk-slave]
driver=com.mysql.jdbc.Driver
user=dbgtest
password=abc@123
host=10.50.1.25
port=3307
dbname=zpTopupLog_BK
```

  

# CSTool

## STG

```Plain
[app]
...
dbBackupThreadHoldDate=20210801

[db-topup-log-bk-slave]
driver=com.mysql.jdbc.Driver
user=dbgtest
password=abc@123
host=10.50.1.25
port=3307
dbname=zpTopupLog_BK
```

  

## REAL

```Plain
[app]
...
dbBackupThreadHoldDate=20210801

[db-topup-log-bk-slave]
driver=com.mysql.jdbc.Driver
user=dbgtest
password=abc@123
host=10.50.1.25
port=3307
dbname=zpTopupLog_BK

[db-topup-staging-log]
driver=com.mysql.jdbc.Driver
user=dbgtest
password=abc@123
host=10.50.1.25
port=3307
dbname=zpTopupLog_BK

[db-topup-staging-log-bk]
user=dbgtest
password=abc@123
host=10.50.1.3
port=3306
dbname=zpTopupLog
```