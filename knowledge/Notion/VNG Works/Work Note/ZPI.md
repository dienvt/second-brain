# zpi-be-billing

## Protocol generate

Clone dependency because get from GO_PATH

`git clone` `[https://dienvt:zZsWq2L8sSRFndejfywQ@gitlab.zalopay.vn/zpi/api/zpi-api-common](https://dienvt:zZsWq2L8sSRFndejfywQ@gitlab.zalopay.vn/zpi/api/zpi-api-common)` `$GOPATH/src/gitlab.zalopay.vn/zpi/api/zpi-api-common`

`git clone` `[https://dienvt:zZsWq2L8sSRFndejfywQ@gitlab.zalopay.vn/zpi/be/zpi-be-notification](https://dienvt:zZsWq2L8sSRFndejfywQ@gitlab.zalopay.vn/zpi/be/zpi-be-notification)` `$GOPATH/src/gitlab.zalopay.vn/zpi/be/zpi-be-notification`

  

move idl to correspond direct in go GO_PATH (because import from this, detail view in .proto file)

`mkdir -p $GOPATH/src/gitlab.zalopay.vn/zpi/be/zpi-be-billing && \ cp idl $GOPATH/src/gitlab.zalopay.vn/zpi/be/zpi-be-billing`

  

install protoc-gen-go

`export GO111MODULE=on && \ go get -u` `[github.com/golang/protobuf/protoc-gen-go@v1.3.4](http://github.com/golang/protobuf/protoc-gen-go@v1.3.4)`

  

get Error

`/Users/lap14211/.gvm/pkgsets/go1.15/global/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis: warning: directory does not exist.`

  

Solution

`go get -u` `[github.com/grpc-ecosystem/grpc-gateway@v1.12.1](http://github.com/grpc-ecosystem/grpc-gateway@v1.12.1)`

change

```Bash

-I$$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
-I$$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway \
```

to

```Bash
-I$$GOPATH/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.12.1/third_party/googleapis \
-I$$GOPATH/pkg/mod/github.com/grpc-ecosystem/grpc-gateway@v1.12.1 \
```

  

  

# zpi-api-billing

## Protocol generate

```Plain
go get -u github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway
go get -u github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2
go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger
go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc
go get -u github.com/golang/protobuf/proto
go get -u github.com/gogo/protobuf/proto
go get -u github.com/golang/protobuf/protoc-gen-go@v1.3.4
```

move idl to GO_PATH

`mkdir -p $GOPATH/src/gitlab.zalopay.vn/zpi/api/zpi-api-billing && cp -rf idl $GOPATH/src/gitlab.zalopay.vn/zpi/api/zpi-api-billing`

  

protoc -I/usr/local/include -I. -I$GOPATH/src -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis --plugin=protoc-gen-grpc-gateway=$GOPATH/bin/protoc-gen-grpc-gateway --grpc-gateway_out=logtostderr=true:. ./helloworld.proto