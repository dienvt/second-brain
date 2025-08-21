


* [ ] create use case, and unit test
* [ ] create gRPC handler
* [ ] update method permission
* [ ] initial integration test file
* [ ] create integration test data new call with many participants in-progress and other status
* [ ] update integration test
* [ ] Implement revoke user Token
	* [ ] Agora: call RTCCommandsGateway.DisconnectParticipant(roomID, userID)
	* [ ] Twilio: the same


Trong cái call thì không hiển thị nữa, nhưng trong cái history call section thì vẫn hiển thị
Trong call thì không hiểu thị user đã được removed
Nhưng trong lịch sử thì vẫn phải hiển thị


Lúc em block user_id bên Agora thì lúc join lại nó đâu cho đâu


check lại flow block user
Tạo token từ userID, nhưng ban thì ban theo user


  
// BuildTokenWithUserAccount Build the RTC token with account.//  
// appId: The App ID issued to you by Agora. Apply for a new App ID from  
//     Agora Dashboard if it is missing from your kit. See Get an App ID.  
// appCertificate: Certificate of the application that you registered in  
//     the Agora Dashboard. See Get an App Certificate.  
// channelName: Unique channel name for the AgoraRTC session in the string format  
// account: The user's account, max length is 255 Bytes.  
// role: RolePublisher: A broadcaster/host in a live-broadcast profile.  
//     RoleSubscriber: An audience(default) in a live-broadcast profile.  
// **tokenExpire: represented by the number of seconds elapsed since now. If, for example,**  
**//     you want to access the Agora Service within 10 minutes after the token is generated,**  
**//     set token_expire as 600(seconds).**  
// privilegeExpire: represented by the number of seconds elapsed since now. If, for example,  
//     you want to enable your privilege for 10 minutes, set privilege_expire as 600(seconds).  
// return The RTC token.





https://webdemo.agora.io/agora-web-showcase/examples/Agora-RTM-Tutorial-Web/
https://webdemo.agora.io/basicVideoCall/index.html

https://webdemo.agora.io/
