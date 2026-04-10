**hỏi về Operation System và Go lang, hỏi về các network transportation protocol, và database. job này làm nặng về xử lý các service của backend.**

**hỏi về luồng (thread, process) của Operation System, hiểu về Database, các kỹ thuật trong database như Indexing, Partitioning. Cả Authentication trong một dự án backend nữa (login, logout)**

OS

Process vs Thread

How many process can create

Context switching

Kernel role

Concurrency:

Mutex and Semaphore

Network

TCP vs UDP

TCP 3-way hand shake

HTTP/HTTPS

Data Structure

Array vs linked list?

Queue and Stack?

Hash map and heap

B-Tree vs B + Tree

Algorithm

Hashing

SHA256

HMAC

RSA

AES

Go programing

Map

Goroutines & Channel

Context

GOMAXPROCS

Industries

JWT token

OAuth2

Server-Side Web Application Flow

Client-Side Web Applications Flow

Resource Owner password

Client Credentials flow

Getting Access to User Data from Mobile Apps

Storage

Indexing

Kafka

Why kafka is fast

Topic, partitions & group

Database

Indexing

# OS

## Process vs Thread

[https://www.youtube.com/watch?v=4rLW7zg21gI](https://www.youtube.com/watch?v=4rLW7zg21gI)

  

Program: **A program is an executable file, It contains the code, or a set of processor  instructions, that is stored as a file on disk. When the code in program is load to memory and be execute by processor → it becomes a process.**

  

An active process also includes the **resources** the program needs to run. These resources are managed by the operating system. Some examples are **processor registers**, **program counters**, **stack pointers**, **memory pages** assigned to the process for its heap and stack, etc. Each process has its **own memory address space**. One process **cannot corrupt** the memory space of another process.

  

So what is a thread?  
A thread is the unit of execution within a process. A process has at least one thread. It is called the main thread. Earlier we mentioned  
**registers, program counters, and stack pointers** as being part of a process. **It is more accurate to say that those things belong to a thread.**

**Threads within a process share a memory address space**, it is possible to communicate between threads using that shared memory space

  

The [**Process Control block(PCB)**](https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/) is also known as a Task Control Block. it represents a process in the Operating System. A process control block (PCB) is a data structure used by a computer to store all information about a process. It is also called the descriptive process. When a process is created (started or installed), the operating system creates a process manager.

  

## How many process can create

[https://www.geeksforgeeks.org/states-of-a-process-in-operating-systems/](https://www.geeksforgeeks.org/states-of-a-process-in-operating-systems/)

![[/Untitled 9.png|Untitled 9.png]]

As for threads, modern CPUs support simultaneous multithreading (SMT), which allows multiple threads to run on a single core. The exact number of threads that can run simultaneously on one core depends on the specific CPU architecture and whether SMT (e.g., Intel's Hyper-Threading or AMD's SMT) is supported. In general, **the number of processes and threads that can effectively run on a CPU core is determined by the CPU's architecture**, the operating system's scheduling algorithms, and the workload being executed.

## Context switching

**Difference between Thread Context Switch and Process Context Switch :**

|No.|Thread Context Switch|Process Context Switch|
|---|---|---|
|0.|There are fewer states to track, and more importantly, since threads share  <br>the same memory address space, there is no need to switch out virtual memory pages,  <br>which is one of the most expensive operations during a context switch|The operating system stores the states of the current running  <br>process so the process can be restored and resume execution at a later point.  <br>It then restores the previously saved states of a  <br>different process and resumes execution for that process|
|1.|TCS occurs when the CPU saves the current state of the thread and switches to another thread of the same process.|PCS occurs when the operating system’s scheduler saves the current state of the running Program (including the state of PCB) and switches to another program.|
|2.|TCS helps the CPU to handle multiple threads simultaneously.|PCS involves loading of the states of the new program for it’s execution.|
|3.|TCS does not involves switching of memory address spaces.All the memory addresses that the processor accounts remain saved.|PCS involves switching of memory address spaces.All the memory addresses that the processor accounts gets flushed.|
|4.|Processor’s cache and Translational Lookaside Buffer preserves their state.|Processor’s cache and TLB gets flushed.|
|5.|Though TCS involves switching of registers and stack pointers, it does not afford the cost of changing the address space.Hence it is more efficient.|PCS involves the heavy cost of changing the address space.Hence it is less efficient.|
|6.|TCS is a bit faster and cheaper.|PCS is relatively slower and costlier.|

## Kernel role

  

  

# Concurrency:

  

## Mutex and Semaphore

In summary, mutexes are designed specifically for exclusive access to a shared resource, ensuring that only one thread can access it at a time, while semaphores are more versatile and can manage access to a resource by multiple threads within a defined limit.

1. **Mutex:**
    - Stands for "Mutual Exclusion."
    - Protects shared resources to ensure that only one thread can access the resource at a time.
    - Typically, it's a binary lock, meaning it has two states: locked or unlocked.
    - When a thread acquires a mutex lock, it gains exclusive access to the resource. Other threads attempting to acquire the lock will be blocked until the lock is released by the owning thread.
    - Mutexes are often used to prevent race conditions in critical sections where data integrity must be maintained.
2. **Semaphore:**
    - A more generalized synchronization primitive that controls access to a shared resource through a counter.
    - Allows a fixed number of threads to access a resource simultaneously.
    - Semaphores can be used for more complex scenarios where multiple threads can access a resource within the limits defined by the semaphore.
    - It maintains a count that represents the number of available resources. When a thread accesses the resource, the count decreases; when it's done, the count increases.
    - Semaphores can be either binary (similar to a mutex) or counting (allowing a specified number of threads to access a resource at the same time).

  

# Network

## TCP vs UDP

|TCP(Transmission Control Protocol)|UDP(User Datagram Protocol)|
|---|---|
|TCP is connection-oriented, meaning it establishes a connection between the sender and receiver before transmitting data. It ensures reliable and ordered delivery of packets.|UDP is connectionless; it does not establish a connection before sending data. It's a fire-and-forget protocol that doesn't guarantee delivery or order of packets.|
|TCP ensures reliability by using acknowledgments, retransmissions, and sequencing of packets. If a packet is lost, it will be retransmitted. This makes TCP suitable for applications that require guaranteed delivery, such as web browsing, email, and file transfer.|UDP does not have built-in mechanisms for error checking, acknowledgment, or packet ordering. Therefore, it's faster but less reliable. It's suitable for real-time applications like video streaming, online gaming, and VoIP where a small amount of packet loss is acceptable.|
|TCP adds additional header information for sequencing, acknowledgment, flow control, and error checking. This makes TCP packets larger and more resource-intensive|UDP has a simpler header structure, resulting in smaller packet sizes and lower overhead. This simplicity makes UDP faster but less robust.|
|TCP ensures that data packets arrive in the same order they were sent. If a packet arrives out of order, TCP will rearrange them before passing them to the application.|UDP does not guarantee packet order. If packets are sent in a certain order, there's no guarantee they will arrive in the same order.|
|TCP is suitable for applications that require reliability and accuracy in data delivery, where retransmission of lost packets is acceptable at the expense of some overhead. It's commonly used for HTTP, FTP, SSH, etc.|UDP does not guarantee packet order. If packets are sent in a certain order, there's no guarantee they will arrive in the same order.|

In summary, TCP provides reliable, ordered, and error-checked delivery at the cost of higher overhead, while UDP sacrifices these features for speed and lower latency, making it suitable for time-sensitive applications where occasional packet loss is acceptable. The choice between TCP and UDP depends on the specific requirements and priorities of the application or service being used.

## TCP 3-way hand shake

Client —— SYN & X → Server

Client ← SYN-ACK & Y & {X + 1} — Server

Client — ACK {Y+1} → Server

  

## HTTP/HTTPS

![[/Untitled 1 2.png|Untitled 1 2.png]]

# Data Structure

## Array vs linked list?

  

## Queue and Stack?

  

## Hash map and heap

  

  

When you insert a key-value pair into a Go map using the `**map[key] = value**` syntax, several things happen internally:

1. **Hashing the Key:** The key is hashed to generate a hash value. This hash value is used to determine the bucket where the key-value pair will be stored in the map.
2. **Finding the Bucket:** Using the hash value, Go determines the bucket where the key-value pair should be placed. The map internally uses a hash table structure consisting of multiple buckets.
3. **Collision Handling:** If there's a collision (i.e., two different keys have the same hash), Go uses a technique called chaining. It stores multiple elements in the same bucket, forming a linked list or a more sophisticated structure if necessary.
4. **Inserting the Key-Value Pair:** Once the correct bucket is found (or created in the case of a new hash), the key-value pair is inserted into the bucket. If the key already exists in the bucket, the value is updated.
5. **Resizing the Map (if necessary):** If the map reaches a certain load factor (a threshold of items per bucket), Go automatically resizes the map, rehashes the keys, and redistributes the elements among the new buckets to maintain efficient access times.
6. **Accessing Elements:** When you later access a value in the map using its key (`**map[key]**`), Go uses the same hash function to find the bucket where the key should be located, then searches or iterates through the elements in that bucket (or the chained list) to find the corresponding value.

It's important to note that Go maps do not guarantee a specific order of elements when iterating over them. The iteration order of map elements can vary between different executions due to the randomized hash function and collision resolution strategies.

## B-Tree vs B + Tree

Add: thêm vào vị trí nếu bị phạm rule 1 < số lượng node < 2*n - 1 thì phải tách

B+Tree chỉ lưu key ở các node ko phải lá, và node lá sẽ link với nhau

  

# Algorithm

  

## Hashing

### SHA256

SHA 256 is a part of the SHA 2 family of algorithms, where SHA stands for Secure Hash Algorithm. Published in 2001, it was a joint effort between the NSA and NIST to introduce a successor to the SHA 1 family, which was slowly losing strength against [brute force attacks.](https://www.simplilearn.com/tutorials/cryptography-tutorial/brute-force-attack)

  

### HMAC

[HMACSHA256](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.hmacsha256?view=net-8.0) is a type of keyed hash algorithm that is constructed from the SHA-256 hash function and used as a Hash-based Message Authentication Code (HMAC). The HMAC process mixes a secret key with the message data, hashes the result with the hash function, mixes that hash value with the secret key again, and then applies the hash function a second time. The output hash is 256 bits in length.

### RSA

- **Public Key** & **Private Key**
- Usage
    - **Encrypt and send.** Using the [public key](https://www.okta.com/identity-101/public-key-encryption/) and an agreed-upon padding scheme, you'll scramble your note and send it along. When the message arrives, the person will use a private key to undo the work and see what's inside
    - **Digital signature**: Sign by private key. The recipient will use the hash value and your public key to reverse the process

### AES

# Go programing

## Map

When you insert a key-value pair into a Go map using the `**map[key] = value**` syntax, several things happen internally:

1. **Hashing the Key:** The key is hashed to generate a hash value. This hash value is used to determine the bucket where the key-value pair will be stored in the map.
2. **Finding the Bucket:** Using the hash value, Go determines the bucket where the key-value pair should be placed. The map internally uses a hash table structure consisting of multiple buckets.
3. **Collision Handling:** If there's a collision (i.e., two different keys have the same hash), Go uses a technique called chaining. It stores multiple elements in the same bucket, forming a linked list or a more sophisticated structure if necessary.
4. **Inserting the Key-Value Pair:** Once the correct bucket is found (or created in the case of a new hash), the key-value pair is inserted into the bucket. If the key already exists in the bucket, the value is updated.
5. **Resizing the Map (if necessary):** If the map reaches a certain load factor (a threshold of items per bucket), Go automatically resizes the map, rehashes the keys, and redistributes the elements among the new buckets to maintain efficient access times.
6. **Accessing Elements:** When you later access a value in the map using its key (`**map[key]**`), Go uses the same hash function to find the bucket where the key should be located, then searches or iterates through the elements in that bucket (or the chained list) to find the corresponding value.

It's important to note that Go maps do not guarantee a specific order of elements when iterating over them. The iteration order of map elements can vary between different executions due to the randomized hash function and collision resolution strategies.

## Goroutines & Channel

Certainly! Goroutines in Go are often referred to as "lightweight threads" due to several reasons:

1. **Low Memory Footprint:** Goroutines are managed by the Go runtime and have a smaller initial stack size compared to traditional threads in other languages. The default stack size for a goroutine is small (a few kilobytes), allowing for the creation of thousands or even millions of goroutines without exhausting system resources. This smaller stack size contributes to their lightweight nature.
2. **Efficient Multiplexing:** Goroutines are multiplexed onto a smaller number of operating system (OS) threads. The Go runtime manages the scheduling of goroutines onto these OS threads, known as the Go scheduler. Multiple goroutines can run concurrently on a smaller number of OS threads, enabling efficient utilization of system resources.
3. **Fast Startup and Context Switching:** Goroutines have a quicker startup time compared to traditional threads. Creating a goroutine is much faster than creating a new OS thread due to their smaller initial stack size and simpler setup. Additionally, switching between goroutines (context switching) is more efficient because it doesn't involve costly OS-level context switches, making them faster than typical thread context switches.
4. **Concurrency with Simplicity:** Goroutines provide a high-level concurrency model that abstracts away many complexities of thread management. They are simple to use and make concurrent programming more approachable compared to explicit thread handling, as they allow developers to focus on the logic rather than managing low-level threading concerns.
5. **Communication via Channels:** Goroutines are designed to communicate and synchronize using channels, providing a safe and efficient means of communication between concurrent tasks. Channels facilitate sharing data and communication between goroutines while avoiding common concurrency issues like race conditions.
6. **Scalability:** The lightweight nature of goroutines enables efficient utilization of available resources, making them suitable for highly concurrent applications such as servers, where managing thousands of concurrent connections efficiently is crucial.

A thread is a path of execution that is scheduled by the operating system to execute the code we write in our functions against a processor. A process starts out with one thread, the main thread, and when that thread terminates the process terminates. **This is because the main thread is the origin for the application**. The main thread can then in turn launch more threads and those threads can launch even more threads.

Goroutines are considered to be **lightweight** because they use little memory and resources plus their initial stack size is small. **Prior to version 1.2 the stack size started at 4K and now as of version 1.4 it starts at 8K**. The stack has the ability to grow as needed.

## Context

`// A Context carries a deadline, a cancellation signal, and other values across// API boundaries.`

`//// Context's methods may be called by multiple goroutines simultaneously.`

## GOMAXPROCS

GOMAXPROCS enable parallel virtual processor can process the goroutines

  

  

# Industries

  

## JWT token

- Header
- Payload
- Signature

Token: ==**Base64(Header)**==.==Base64(Payload)==.==(Header.Alg(base64UrlEncode(header)+"."+ base64UrlEncode(payload))==

  

ex:

```JSON
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

## OAuth2

### Server-Side Web Application Flow

`state` - A unique value used by your application in order to prevent  
cross-site request forgery (CSRF) attacks on your implementation.  
The value should be a random unique string for this particular  
request, unguessable and kept secret in the client (perhaps in a  
server-side session)  

Get Code → exchange for token → refresh token

```Bash
$queryParams = array(
  'client_id' => '240195362.apps.googleusercontent.com',
  'redirect_uri' => (isset($_SERVER['HTTPS'])?'https://':'http://') .
                   $_SERVER['HTTP_HOST'] . $redirectUriPath,
  'scope' => 'https://www.googleapis.com/auth/tasks',
  'response_type' => 'code',
  'state' => $_SESSION['state'],
  'approval_prompt' => 'force', // always request user consent
  'access_type' => 'offline' // obtain a refresh token
);
```

  

### Client-Side Web Applications Flow

Same as server-side but access-token will be sent in URI. For example `[http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600](http://photoviewer.saasyapp.com/pv/oauth2callback.html#access_token=ya29.AHES6ZSzX&token_type=Bearer&expires_in=3600%E2%80%9D)`

  

### Resource Owner password

```Bash
curl -d "grant_type=password" \
-d "client_id=3MVG9QDx8IKCsXTFM0o9aE3KfEwsZLvRt" \
-d "client_secret=4826278391389087694" \
-d "username=ryan%40ryguy.com" \
-d "password=_userspassword__userssecuritytoken_" \
https://login.salesforce.com/services/oauth2/token
```

  

### Client Credentials flow

“There is another representative case for the Client Credentials  
flow—when a resource owner has granted an application access to their  
resources out of band, without using a typical OAuth flow. Google provides  
a concrete use case in the Google Apps Marketplace. When an application is listed  
on the Marketplace, vendors get credentials that represent their  
application and also register the scopes of data they need access to. When  
the application is later installed by an organization’s IT administrator,  
Google asks the administrator whether it’s OK to grant the application  
access to his organization’s data. When access is approved, Google stores  
that organization “Acme Corp” has granted access to “Google Calendar and  
Google Contacts” for application “Task Manager Pro.” Google does not issue  
any tokens to the application. When the application tries to access data  
in the future, Google simply looks up whether the application is allowed  
access to data for the particular organization.”  
Like mod post bài và admin có thể sửa  

`client_id` and `client_secret` & `grant_type="client_credential"`

  

### Getting Access to User Data from Mobile Apps

  

# Storage

  

## Indexing

While indexing offers numerous advantages in relational database management systems (RDBMS), it also comes with some drawbacks:

1. **Overhead in Insertions, Updates, and Deletions**: Whenever data is inserted, updated, or deleted in a table with indexes, the database must also maintain the indexes. This maintenance overhead can slow down these operations, especially on tables with multiple indexes.
2. **Increased Storage Requirements**: Indexes consume additional storage space. For large tables or tables with many indexes, the storage overhead can be substantial. This can lead to increased disk space usage and possibly additional hardware costs.
3. **Impact on Performance in Some Operations**: While indexes speed up retrieval operations (such as SELECT queries), they might slow down other operations. For instance, too many indexes on a table can negatively affect the performance of insertions, updates, and deletions due to the maintenance overhead mentioned earlier.
4. **Risk of Outdated Statistics**: The database optimizer uses statistics to determine the best execution plan for queries. Outdated statistics on indexes can mislead the optimizer, leading to suboptimal query execution plans and reduced performance.
5. **Complexity in Management and Maintenance**: As the number of indexes grows, managing and maintaining them becomes more complex. Regular monitoring, index tuning, and ensuring the relevance of indexes become essential but can be time-consuming tasks.
6. **Potential for Index Fragmentation**: Over time, indexes can become fragmented due to insertions, updates, and deletions. Fragmentation can impact performance and might require periodic index maintenance operations to reorganize or rebuild indexes.
7. **Not Suitable for All Types of Queries**: While indexes significantly speed up certain types of queries, they might not benefit all queries equally. Some queries might not use indexes effectively, leading to no performance improvement or even performance degradation.
8. **Limitations in Some Database Systems**: Certain database systems have limitations on the number or size of indexes that can be created on a table, which can restrict the ability to optimize performance through indexing.

To mitigate these disadvantages, it's crucial to carefully plan and selectively create indexes based on the actual usage patterns of the database. Regular performance monitoring, index maintenance, and keeping statistics up-to-date are essential practices to ensure that indexes continue to benefit query performance without causing unnecessary overhead.

# Kafka

## Why kafka is fast

- Sequence read write to disk
- Zero copy from CPU to socket

  

## Topic, partitions & group

  

  

# Database

  

## Indexing

  

  

Fundamental:

Process vs Thread: understand the concept. Know process and thread sharing memory.  
Context switch: doesn't know  
Network: TCP/UDP know the difference but only list TCP use package sequence number  
Concurrent: Doesn't know Mutex  
Array vs Linked List: know vaguely the concept  
Industrial Practice:  

Authen vs Author: ok  
JWT Token: know the concept, but cannot explain clearly how JWT ensure authen and author  
Database: Index: know benefit. Disadvantages, can only list resource-consuming, but doesn't mention memory  
Index data structure: know Hash, doesn't know b+Tree  
Kafka: doesn't explain correctly consumer group, partition"  

"Fundamental system:

OS thread vs process: know process consisting of many threads, sharing variables but mistake in number of processes capped by the number of CPU cores.  
Goroutines: know goroutines are light-weight but doesn't know why  
Context switching: (1/5) know vaguely that context switching is for switching some pointers when switching between threads. But cannot clearly list detail what to be done for context switching  
Array vs Linked List: know the definition, can explain complexity. Cannot explain complexity of append to an array when reaching size limit  
TCP vs UDP: only knows TCP is connection-oriented and UDP is connection-less.  
Industry practices:  

JWT Tokens: know a token contains some data and somewhat encrypted. But doesn't explain the mechanism  
Database Index: know what index is and mention B-Tree, but doesn't explain what BTree is  
Composite Index: know the the differences in column order of a composite index but give a wrong example where the index is not used."