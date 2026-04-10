# Bài test:

Nghiệp vụ thanh toán hoá đơn của ZaloPay đứng ra kết nối giữa người cần thanh toán hoá đơn (user) và phía cung cấp dịch vụ thanh toán hoá đơn (provider). Giả sử bạn được giao cho việc thiết kế hệ thống giúp điều phối request thanh toán hoá đơn từ user đến provider.

Yêu cầu:

- Có thể change tỉ lệ request sang các provider realtime
- Có thể scale lên nhiều node

Bạn được giao nhiệm vụ viết một service load balancing chịu trách nhiệm cho việc route các request từ user vào các service instance của zalopay.

Hệ thống ZaloPay đang có dự định

  

Hiện tại ZaloPay đang deploy các service của mình trên nhiều server vật lý, mỗi server vật lý sẽ có một địa chỉ ip để gọi vào. Để tuỳ chỉnh số lượng request vào các server theo một tỉ lệ cụ thể, ZaloPay đang xây dựng một service trung gian chị trách nhiệm điều phối request. Giả sử bạn được phân công thiết kế và hiện thực service trên, bạn sẽ hiện thực nó như thế nào?

Service trên cần có những chức năng:

- Input một domain và nhận về một địa chỉ ip (đại diện cho một server vật lý).
- Có thể config được domain nào sẽ được route vào **những** địa chỉ ip nào, với tỉ lệ bao nhiêu.
- Config có thể được thay đổi real time.
- Có thể deploy nhiều instance.

Để nộp bài thì bạn vui lòng publish source trên github.

  

Viết một service load balancing điều phối request vào từng

Những service của ZaloPay hiện đang chỉ được deploy một instance và việc này dẫn tới việc nhiều chức năng sẽ bị tắt khi tiến hành deploy. Trong tương lai gần ZaloPay sẽ deploy các service của mình trên nhiều instance, để tuỳ chỉnh số lượng request vào các instance trên sẽ cần có một service DNS. Khi một request gọi vào gateway của ZaloPay, gateway sẽ gọi vào DNS với thông tin domain và nhận lại một ip đại diện cho instance xử lý request trên. của instance được này sẽ nhận vào một string param đại diện cho domain request đó cần truy cập. có một service mang tên load balancing (LB). LB sẽ đóng vai trò như một DNS giúp phân giải domain sang địa chỉ IP của từng instance. Giả sử bạn được phân công viết service LB nêu trên bạn sẽ hiện thực nó như thế nào?

Tiêu chí đánh giá:

- có thể tuỳ chỉnh tỉ lệ request vào từng instance real time
    - có thể scale service LB lên nhiều nodes

  

# Chào hỏi:

- Giới thiệu mình là ai, đang làm gì. anh là Điền hiện đang dev và quản lý sản phầm thanh toán hoá đơn của app ZLP.
- Giới thiệu anh Vũ là ai và đang làm gì.

# Phần đặt câu hỏi:

Hỏi về project đã làm thường là cũ nhất? visualize kiến thức lên bảng

  

Project nào tâm đắc

có làm pet project hay không?

Thuật toán random

hỏi về locking

Hỏi về database cũng không trả lời được

Hỏi về queue

Bạn

  

---

Mai thiên Phú:

Migrate

bảo hiểm

java:

transaction a.c.i.d

transaction s.o.l.i.d

  

chia team theo business logic: người được nhận

Giao tiếp :

- rabbitMQ: nhận message đẩy ra ngoài CDC. Dùng
- Grpc & RestAPI :
    - Grpc: call một service. http2
    - RestAPI: http2

  

DB schema:

A.C.I.D

fronted làm được

OOP:

  

Design pattern:

- strategy
- builder
- adapter
- factory
- abstract factory

  

SOLID:

- D: cũng ok

  

multi thread

l ambd a +.

stream

  

tham trị

  

DNS ⇒ string (IP)

  

IP1 IP2

1k → IP1 : IP2

map[dns][IP1,IP2]

  

IP1, IP2.

3:7

→ count +

→ total đạt rồi , →

30 →

  

→

  

  

  

  

hỏi nhà:

hỏi khi nào làm được: 10 tháng 10

hỏi khi nào

---

  

  

  

Không gây ấn tượng lắm

2 năm 2020 spring, hibernate,

  

grpc

  

rabbitMQ noti

cdc

etag accepted

  

đóng gói

ab.

kế thừa.

đa hình.

  

tứ giác

  

  

first core service

  

java app

call connect

giao thuc chung: rest, grpc.

resp des, bina

BA: làm

sql, maria

transaction a.c.i.d

transaction s.o.l.i.d

parrallell

CDC

set

cache config

dep cross

  

collection

  

Feedback:

- Về mức am hiểu về hệ thống đang làm: chưa thực sự hiểu rõ
- Kiến thức về hạ tầng
- Xử lý multi thread cũng chưa được mượt

Bạn Khoa hầu như không đáp ứng được nhu cầu của team (ở vị trí software engineer). Anh có hỏi nhiều câu về việc thiết kế hệ thống cũng như kiến thức về hạ tầng nhưng câu trả lời của bạn không đạt được mong đợi của anh.

Mặc khác thì đặc trưng của team là làm hệ thống phân tán và nhiều luồng nhưng mà bạn không mạnh ở cả hai khoản trên.

Có hỏi thêm một số câu hỏi về định hướng nhưng mà những

  

Kiến thức về hạn tầng chưa đáp ứng đủ, detail là về database cũng như các thư viện đang sử dụng.

  

Process vs Thread?

- Multi-threading
- Parallel vs Concurrent vs Asynchronous

Language

abstraction vs interface

final class, final variable

Frame

DI vs AOP và AOI

web trading:

- order securities là làm cái gì trong đó
- Deposit với withdraw khi có lỗi thì làm thế nào
- Transfer đang làm thế nào

  

## Java

Con trỏ của java, tham chiếu hay tham trị

Link list khác gì array list

abstraction vs interface

final class, final variable

SOLID

DI vs AOP và AOI

  

## E-commerce

Load balance

web trading:

- order securities là làm cái gì trong đó
- Deposit với withdraw khi có lỗi thì làm thế nào
- Transfer đang làm thế nào

Process vs Thread?

- Multi-threading
- Parallel vs Concurrent vs Asynchronous

# Tool

git vs github vs gitlab

Phải lock đúng không ? hỏi về Mutex vs Semaphore

Mutex and semaphore are synchronization mechanisms used in concurrent programming, but they serve different purposes:

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

In summary, mutexes are designed specifically for exclusive access to a shared resource, ensuring that only one thread can access it at a time, while semaphores are more versatile and can manage access to a resource by multiple threads within a defined limit.

## Internet

  

  

# Nam

[`github.com/go-delve/delve/cmd/dlv`](http://github.com/go-delve/delve/cmd/dlv) vaf [`github.com/cosmtrek/air`](http://github.com/cosmtrek/air) là gì?

  

xoá cart đã thanh toán được không?

Sao lúc tạo cart lại set `cart.Product = nil`?

  

tại sao cái `cart.Product.Price` lại là Product ???

  

Ko xử lý data race