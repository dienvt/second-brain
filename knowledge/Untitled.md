||v1.4.0|v2.1.0|
|---|---|---|
||Execute:||
|PostMsgReceive|None|PostMsgReceive|
|PostMsgSend|messageFunc|PostMsgSend|
|PostCall|messageFunc|PostCall|
|Skip PostMsgReceive|shouldLog|LoggableEvent : StartCall|
|Skip PostMsgSend|shouldLog|????|
|Skip PostCall|shouldLog|????|
|ErrorToCode|DefaultErrorToCode try to convert to GrpcCode||
|CodeToLevel|// DefaultCodeToLevel is the default implementation of gRPC return codes and interceptor log level for server side.||
|func DefaultCodeToLevel(code codes.Code) zapcore.Level {|||

```
switch code {
case codes.OK:
	return zap.InfoLevel
case codes.Canceled:
	return zap.InfoLevel
case codes.Unknown:
	return zap.ErrorLevel
case codes.InvalidArgument:
	return zap.InfoLevel
case codes.DeadlineExceeded:
	return zap.WarnLevel
case codes.NotFound:
	return zap.InfoLevel
case codes.AlreadyExists:
	return zap.InfoLevel
case codes.PermissionDenied:
	return zap.WarnLevel
case codes.Unauthenticated:
	return zap.InfoLevel // unauthenticated requests can happen
case codes.ResourceExhausted:
	return zap.WarnLevel
case codes.FailedPrecondition:
	return zap.WarnLevel
case codes.Aborted:
	return zap.WarnLevel
case codes.OutOfRange:
	return zap.WarnLevel
case codes.Unimplemented:
	return zap.ErrorLevel
case codes.Internal:
	return zap.ErrorLevel
case codes.Unavailable:
	return zap.WarnLevel
case codes.DataLoss:
	return zap.ErrorLevel
default:
	return zap.ErrorLevel
}
```

} | | | Inject field | Using ctxzap | Have to write or using [http://github.com/grpc-ecosystem/go-grpc-middleware/v2/interceptors/logging](http://github.com/grpc-ecosystem/go-grpc-middleware/v2/interceptors/logging) | | Difff | "system": "grpc | "protocol": "grpc" | | | "span.kind": "server" | "grpc.component": "server" | | | | | | | | |