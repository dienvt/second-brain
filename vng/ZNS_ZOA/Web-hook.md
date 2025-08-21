# Visualize

```plantuml
 
title "Query"

box "ZaloPay" #LightGreen
participant "Web-hook receiver" as receiver
participant "ZNS trigger" as trigger
end box

box "infoBip"
participant Fowarder as fwd
end box

box "Zalo" #LightBlue
participant "Sender" as sender
participant "Follow web-hook" as follow
participant "ZNS web-hook" as zns
end box

== User follow event ==
follow -> fwd: user_id
fwd -> receiver: user_id

== ZNS trigger ==
trigger -> fwd: phone, msg_id_1
fwd -> sender: phone, msg_id_2
zns -> fwd: phone, msg_id_2
fwd -> receiver: msg_id_1
```


