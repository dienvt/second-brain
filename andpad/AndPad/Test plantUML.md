
```plantuml
@startmindmap
* root topic
** child topic
*** grand topic 1
*** grand topic 2
*** grand topic 3
** child topic 2
*** grand topic 21
@endmindmap
```

```plantuml
@startuml
participant Notification as sns
participant Mobile as mb
participant RemoteBFF as bff
participant Remote as rm

sns -> mb: Incoming call with session_id
mb -> bff: Get latest session 
bff -> rm: Get session
rm -> bff: session entity
bff -> mb: session entity with room_id

mb -> bff: Call `accept_video_call` with room_id
bff -> rm: Grpc acceptVideoCall with room_id
rm -> bff: Return RTC/RTM token 
bff -> mb: Return RTC/RTM token

@enduml
```

Case user start call by new API, then incoming call to the old app. **Old app will call to get room_id from session** so where do we get room_id? 
+ Generate if there are no room_id yet?
+ Change room_id by session_id in old  flow? so how do we know when it be room_id?


Mobile chỉ cần cái 
-> Session hasn't had room_id