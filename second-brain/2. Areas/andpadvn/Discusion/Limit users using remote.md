Fetch profiles
Cần check thêm user có được sử dụng tính năng này hay không?
user has no permission thì nên đá ra hay là hiện lỗi. BE trả về lỗi sau đó đá ra login.

[admin] Thêm số lượng member được sử dụng remote (config)
\[profile\]\[user\]  
\[] Error message 
- Trong trang admin enable remote và setup số lượng member được quyền sử dụng remote. Phải check số lượng người sử dụng andpad
- List permission trả về fail
- Call những API khác trả về 403 còn lại FE quyết định.


\[user setting] add view that admin can see 
\[builder pad admin] enable and add user to whitelist
\[remote admin] show how many user can add and current user has been grant permission
\[] flow oauth của bên web đang như thế nào? tại nếu ko có permission thì nó sẽ ko auth được.

ListJoinClient,
Thường thì BFF sẽ phải handle trường hợp empty -> trả ra lỗi hay là FE sẽ phải xử lý.


RTCProvider 