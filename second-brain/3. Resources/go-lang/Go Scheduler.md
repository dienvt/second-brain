---
tags:
  - go-lang
---
![[Drawing 2025-06-15 19.57.19.excalidraw]]


There are two types of work a Thread can do. The first is called CPU-Bound and the second is called IO-Bound.

**CPU-Bound**: This is work that never creates a situation where the Thread may be placed in Waiting states. This is work that is constantly making calculations. A Thread calculating Pi to the Nth digit would be CPU-Bound. The thread is intensive in terms of CPU operations, then it is called CPU bound process.

**IO-Bound**: This is work that causes Threads to enter into Waiting states. This is work that consists in requesting access to a resource over the network or making system calls into the operating system. A Thread that needs to access a database would be IO-Bound. I would include synchronization events (mutexes, atomic), that cause the Thread to wait as part of this category. The process is intensive in terms of I/O operations then it is called I/O bound process.
