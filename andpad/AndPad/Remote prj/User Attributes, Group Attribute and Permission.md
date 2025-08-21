# Structure

Hiện tại remote đang đi theo client (công ty) chứ ko đi them order (dự án)
 + User's client_id sẽ là đại diện cho việc user đó đang thuộc công ty chủ quản nào
 * Group's client_id sẽ là đại diện cho việc user đó đang thuộc công ty chủ quản nào

```
message UserAttributes {  
  int64 user_id = 1;   // @gotags: validate:"gt=0"  
  int64 client_id = 2; // @gotags: validate:"gt=0"  
}
```

```
message GroupAttributes {  
  string group_id = 1;         // @gotags: validate:"required"  
  int64 client_id = 2;         // @gotags: validate:"gt=0"  
  optional int64 order_id = 3; // @gotags: validate:"omitempty,gt=0"  
}
```


