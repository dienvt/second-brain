```
@startuml
abstract Communication
Communication : Comunicate()

class Chat
class Message
class VideoCall

class UI
UI : InputPhoneNumber()
UI : InputMessage()
UI : ShowMessage()
UI : OpenCamera()
UI : OpenMic()


Communication <|-- Chat
Communication <|-- Message
Communication <|-- VideoCall


Chat o-- UI
Message o-- UI
VideoCall o-- UI
@enduml
```


```
@startuml

abstract Communication {
  Comunicate()
}

together {
  class Call
  class Message
  class VideoCall
}

class UI
UI : InputPhoneNumber()
UI : InputMessage()
UI : ShowMessage()
UI : OpenCamera()
UI : OpenMic()

interface CallUI
CallUI : InputPhoneNumber()
CallUI : OpenMic()

interface MessageUI
MessageUI : InputPhoneNumber()
MessageUI : InputMessage()

interface VideoCallUI
VideoCallUI : InputPhoneNumber()
VideoCallUI : OpenCamera()
VideoCallUI : OpenMic()

CallUI - VideoCallUI

Call o-- CallUI
Message o-- MessageUI
MessageUI -up[hidden]-> Message
VideoCall o-- VideoCallUI

Communication <|-- Call
Communication <|-- Message
Communication <|-- VideoCall

CallUI <|-- UI
MessageUI <|-- UI
VideoCallUI <|-- UI
@enduml
```