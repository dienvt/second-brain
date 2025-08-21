https://88-oct.atlassian.net/wiki/spaces/OCT/pages/3086222206/Support+dual+RTC+Providers+Agora+and+Twilio

Lúc accept call thì cần truyền X-RTC-PROVIDER
Vì trường hợp call AcceptCall thì phía Server sẽ verify ok và trả về agora token. Tuy nhiên là client version cũ sẽ bị lỗi vì không support Agora.
Trong TH đó thì cần chặn trước lúc call accept call, client phải gửi thông tin provider lên.

Nhưng mà phía client support cả 2 thằng thì sao? làm sao để client biết mà truyền lên đúng `X-RTC-PROVIDER`

Update UI,
Lúc get client bị lỗi thì nên hiện lỗi hay là cứ thử start call bằng twilio

Xuất phát từ việc trả thêm RTCProvider trong session thì FE có suggest tại sao FE phải cần truyền header `X-RTC-PROVIDER`, BE có thể tự quyết định và truyền về trong response. Sau đó, FE (mobile, web) sẽ tự switch component dựa trên thông tin `RTCProvider` trong response.

Lúc này nảy sinh 2 solutions:
- FE (mobile, web) không cần truyền header `X-RTC-PROVIDER` nữa. Bị một vấn đề là trong trường hợp Client bật enable **Agora**  những app version cũ thì UX tệ (hiện loading liên tục vì dùng SDK **Twilio** để load token **Agora**). Tuy nhiên là Web sẽ không cần phải call check Client Permission (không hợp lý)
- FE (mobile, web) vẫn truyền `X-RTC-PROVIDER`. BE sử dụng `X-RTC-PROVIDER` có thể phân biệt được app cũ hay là app mới (ko make sense)

Dùng version không được vì dùng API có thể giả mạo version

RTC provider sẽ có type string **ko define enum** 

call permission bij lỗi, return message cho user. BE trả về message để
Internal sever, bad request thì FE sẽ quyết định
BE xử lý lỗi thì FE sẽ dùng chính message đó để show 
sau đó sẽ dừng.
