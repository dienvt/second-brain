
## Use case
Dùng để check khi tạo group, cần truyền list user và client_id

## endpoint
/clients/{clientid}/users/remote_enable


There is two main table
- remote_call_client_settings
	- 

check lại chỗ member đang ntn
Check lại remote đang call ntn

call internal nên ko cần authen
truyền userID trong header
internal/remote/clients
> Check chỗ list client làm sao mà list ra được


response về thì thường là map\[userID]: enable hay không
`members`
ClientID, GroupID, UserID

Luôn có httpReq.Header.Set("X-ANDPAD-USER-ID", request.UserID.String())
=> Suy ra là luôn phải login bằng cách set header `X-ANDPAD-USER-ID`

Create group đang call workman để check user 
```go
members, err := s.workmanQueriesGateway.BatchGetMembersByUserID(  
    ctx,  
    tenant.TenantID(in.ClientID),  
    in.UserIDs,  
)
```
