---
title: "hostname"
date: 2026-01-14
tags:
  - engineering
  - docker
  - devops
---

docker host name is like cluster name 

create network then connect those containers with each others


There are many network drive we can use

|Driver|Description|
|---|---|
|`bridge`|The default network driver.|
|`host`|Remove network isolation between the container and the Docker host.|
|`none`|Completely isolate a container from the host and other containers.|
|`overlay`|Overlay networks connect multiple Docker daemons together.|
|`ipvlan`|IPvlan networks provide full control over both IPv4 and IPv6 addressing.|
|`macvlan`|Assign a MAC address to a container.|

ref:
https://docs.docker.com/network/