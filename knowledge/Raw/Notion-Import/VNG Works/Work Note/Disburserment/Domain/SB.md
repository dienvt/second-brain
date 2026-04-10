---
domain: http://sbgrpc-disbursement.zpapps.vn/
"IP: PORT": 10.109.3.86:8081
---
93.9:3306

  

## Docker run

```Bash
// core service
docker run -d --name=disbursement-core -v /app/disbursement-core:/app -p 8080:8080 8081:8081 --restart=always acq-go:0.1 ./srv

// cron service
docker run -d --name=disbursement-cron -v /app/disbursement-core:/app -p 8082:8082 --restart=always acq-go:0.1 ./srv cron

docker run -d --name=disbursement-core-lt \\
	--add-host ltgrpc-td-disbursement.zalopay.vn:10.60.45.64 \\
	--add-host lttpe-grpc.zalopay.vn:10.60.45.64 \\
	--add-host grpc-ltuprofile.zalopay.vn:10.60.45.64 \\
	--add-host ltgrpc-zas.zalopay.vn:10.60.45.64 \\
	-v /app/lt/disbursement-core:/app \\
	-p 8910:8080 -p 8911:8081 \\
	--restart=always basego:0.3 ./srv

/////// loadtest
docker run -d --name=disbursement-core-lt \\
--add-host ltgrpc-td-disbursement.zalopay.vn:10.60.45.64 \\
--add-host lttpe-grpc.zalopay.vn:10.60.45.64 \\
--add-host grpc-ltuprofile.zalopay.vn:10.60.45.64 \\
--add-host ltgrpc-zas.zalopay.vn:10.60.45.64 \\
--add-host internal-socialdev.zalopay.vn:10.30.83.19 \\
-v /app/lt/disbursement-core:/app \\
-p 8910:8080 -p 8911:8081 \\
--restart=always basego:0.1


# sandbox qc
docker run -d --name=disbursement-core \
--add-host qcgrpc-td-disbursement.zalopay.vn:10.40.81.10 \
--add-host qcgrpcpe-adapter.zalopay.vn:10.40.81.10 \
--add-host qcgrpc-user-profile.zalopay.vn:10.40.81.10 \
--add-host internal-socialdev.zalopay.vn:10.30.83.19 \
--add-host grpc-sbapimep-private.zpapps.vn:10.50.32.32 \
-v /app/disbursement-core:/app \
-p 8080:8080 -p 8081:8081 \
--restart=always cpsgo:0.3
```