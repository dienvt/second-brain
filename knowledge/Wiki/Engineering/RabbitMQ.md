---
title: "**Note**"
date: 2024-01-01
tags:
  - engineering
  - rabbitmq
---

# **Note**

- **Management page:** [http://localhost:15672/](http://localhost:15672/)
- Khi comsumer nhận được message -> xử lý sau đó thông báo lại cho queue bằng

```Plain
long deliveryTag = envelope.getDeliveryTag();
channel.basicAck(deliveryTag, true);
```

hoặc

```Plain
channel.basicReject(deliveryTag, true);
```

tự độngtạo exchange trên rabbitmq

`channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);`

Dùng JSON format trước khi send vào queue

=> so sánh hiệu năng.

# **Đặt vấn đề**

Trong một hệ thống phân tán, có rất nhiều thành phần và những thành phần này sẽ luôn có nhu cầu giao tiếp với nhau. Nếu như hai thành phần cần giao tiếp với nhau, chúng ta tạo một kết nối trực tiếp thì khi số lượng thành phần tăng lên, thì số lượng kết nối cũng tăng dẫn tới khó khăn trong việc phát triều, debug và maintain.

Giải pháp cho vấn đề trên là thay thế các kết nối trực tiếp bằng hệ thống message broker. Producer sẽ gửi message vào queue hoặc exchange, topic,… và Consumer chỉ cần subcribe vào queue hoặc topic để nhận message.

Bài viết này sẽ là bài giới thiệu về RabbitMQ.

# **Rabbitmq là gì?**

> RabbitMQ is the most widely deployed open source message broker.

Rabbitmq là một mesage broker được lập trình bằng ngôn ngữ Erlang và nó hỗ trợ 2 pattern chính của messaging là message queuing và publish/subscribe. Rabbitmq hỗ trợ các giao thức AMQP, STOMP, MQTT, HTTP và WebSockets.

# **Rabbitmq hoạt động như thế nào?**

Để cho dễ hình dung thì chúng ta sẽ chia quá trình gửi một Message từ producer tới Consume thành 3 giai đoạn: publishing, routing và consuming.

Trước khi đi vào tìm hiểu cách rabbitmq hoạt động, mọi người cần nắm một số khái niệm như sau:

- **Producer:** Chương trình gửi messages.
- **Consumer:** Chương trình nhận messages.
- **Message:** Thông tin được gửi và nhận thông qua message brocker. Mỗi message sẽ có nhiều attribute trong đó có một số cái quan trọng (Content type,Content encoding và Routing key)Ngoài ra còn có các khái niệm khác sẽ được giải chi tiết ở phần sau.

## **1. publishing**

Đây là quá trình Producer gửi message vào hệ thống message broker.

Đầu tiên Producer sẽ tạo connection đến rabbitMQ, đây là một long-lived connection. Trên connection này Producer có thể tạo nhiều channel( session), mỗi channel sẽ được phân biết với nhau thông qua một ID và việc gửi/nhận message sẽ được thực hiện trên channel (session). Lưu ý là channel chỉ tồn tại trên một connection vì vậy nếu connection bị đóng, toàn bộ channel được tạo trên nó cũng sẽ bị đóng.

Exchange sẽ nhận message từ Producer và routing message đó vào các queue tương ứng. Một Exchange được khai báo sẽ bao gồm nhiều thuộc tính, trong đó những thuộc tính quan trọng nhất là exchange type, name, durability, auto-delete, arguments.

## **2. Routing**

Exchange không lưu trữ message cho nên nó phải chuyển message đó tới Queue, nơi sẽ lưu message và đợi Consume tới lấy.

Một Queue phải bao gồm các thuộc tính: Name, Durable, Exclusive, Auto-delete, Arguments.

Note:

- Để sử dụng 1 queue, thì nó phải được khai báo. Nếu queue chưa tồn tại, nó sẽ được tạo. Nhưng nếu queue đã tồn tại thì việc khai báo sẽ không có tác dụng, nhưng nếu việc khai báo không trùng với thông tin queue đã tạo thì exception with code 406 (PRECONDITION_FAILED) will be raised.
- Queue name không thể trùng với pattern “amq.”

Vậy làm sao Exchange biết phải routing message tới queue nào?

Điều kiện cần là Exchange phải được kết nối (bound) với Queue. Sau đó việc message có được routing đến queue hay không phụ thuộc vào 2 yếu tố: Exchange Type và bindlings( bộ quy tắt Exchange sử dụng trên message routing key và message header )

|   |   |   |
|---|---|---|
|**Exchange type**|**Default pre-declared names**|**Mô tả**|
|Direct exchange|(Empty string) and amq.direct|Message sẽ được routing vào queue khi vào chỉ khi routing key giữa Exchage và Queue trùng với routing key trong message|
|Fanout exchange|amq.fanout|Mọi Queue bound với Exchange đều nhận được message.|
|Topic exchange|amq.topic|Exchange và queue sẽ bound với nhau thông qua một pattern, nếu routing key trong message khớp với pattern trên thì message sẽ được routing vào queue|
|Headers exchange|amq.match (and amq.headers in RabbitMQ)|plugin của rabbitmq|

Note: Nếu một message không thể routing tới bất kì queue nào, thì nó sẽ được xóa hoặc trả về cho publisher, phụ thuộc vào messages attribute mà Publisher đã set

If a message cannot be routed to any queue (for example, because there are no bindings for the exchange it was published to) it is either dropped or returned to the publisher, depending on message attributes the publisher has set.

## **3. Consuming**

Consumer có thể lấy được message thông qua 2 cách:

- push API: Subcribe vào queue để nhận được message.
- Pull API:

Storing messages in queues is useless unless applications can consume them. In the AMQP 0-9-1 Model, there are two ways for applications to do this:

Subscribe to have messages delivered to them (“push API”): this is the recommended option

Polling (“pull API”): this way is highly inefficient and should be avoided in most cases

With the “push API”, applications have to indicate interest in consuming messages from a particular queue. When they do so, we say that they register a consumer or, simply put, subscribe to a queue. It is possible to have more than one consumer per queue or to register an exclusive consumer (excludes all other consumers from the queue while it is consuming).

Each consumer (subscription) has an identifier called a consumer tag. It can be used to unsubscribe from messages. Consumer tags are just strings.

The routing algorithm used depends on the exchange type and rules called bindings. AMQP 0-9-1 brokers provide four exchange types:

Đây là giai đoạn xảy ra trong nội tại rabbitmq. Message sẽ được chuyển từ Exchange đến Queue thông qua routing key và header.

Queue chứa message cái mà sẽ được comsume bởi Consumer. ngoài ra một queue phải bao gồm các thuộc tính: Name, Durable, Exclusive, Auto-delete, Arguments.

Note:

- Để sử dụng 1 queue, thì nó phải được khai báo. Nếu queue chưa tồn tại, nó sẽ được tạo. Nhưng nếu queue đã tồn tại thì việc khai báo sẽ không có tác dụng, nhưng nếu việc khai báo không trùng với thông tin queue đã tạo thì exception with code 406 (PRECONDITION_FAILED) will be raised.
- Queue name không thể trùng với pattern “amq.”
- **Queue:** Hàng đợi chứa messages.
- **Message:** Thông tin được gửi và nhận thông qua hệ thống RabbitMQ.
- **Connection:** Kết nối từ chương trình tới hệ thống RabbitMQ broker.
- **Channel:** Kết nối ảo nằm trên Connection, mọi thao tác gửi nhận đều được thực hiện trên kết nối này.
- **Exchange:** Receives messages from producers and pushes them to queues depending on rules defined by the exchange type. In order to receive messages, a queue needs to be bound to at least one exchange.
- **Binding:** Kết nối giữa Exchange và Queue.
- **Routing key:** The routing key is a key that the exchange looks at to decide how to route the message to queues. The routing key is like an address for the message.
- **AMQP:** AMQP (Advanced Message Queuing Protocol) is the protocol used by RabbitMQ for messaging.
- **Users:** It is possible to connect to RabbitMQ with a given username and password. Every user can be assigned permissions such as rights to read, write and configure privileges within the instance. Users can also be assigned permissions to specific virtual hosts.
- **Vhost, virtual host:** A Virtual host provides a way to segregate applications using the same RabbitMQ instance. Different users can have different access privileges to different vhost and queues and exchanges can be created so they only exist in one vhost.

Connections

AMQP 0-9-1 connections are typically long-lived. AMQP 0-9-1 is an application level protocol that uses TCP for reliable delivery. Connections use authentication and can be protected using TLS. When an application no longer needs to be connected to the server, it should gracefully close its AMQP 0-9-1 connection instead of abruptly closing the underlying TCP connection.

Channels

Some applications need multiple connections to the broker. However, it is undesirable to keep many TCP connections open at the same time because doing so consumes system resources and makes it more difficult to configure firewalls. AMQP 0-9-1 connections are multiplexed with channels that can be thought of as “lightweight connections that share a single TCP connection”.

Every protocol operation performed by a client happens on a channel. Communication on a particular channel is completely separate from communication on another channel, therefore every protocol method also carries a channel ID (a.k.a. channel number), an integer that both the broker and clients use to figure out which channel the method is for.

A channel only exists in the context of a connection and never on its own. When a connection is closed, so are all channels on it.

For applications that use multiple threads/processes for processing, it is very common to open a new channel per thread/process and not share channels between them.

Virtual Hosts

To make it possible for a single broker to host multiple isolated “environments” (groups of users, exchanges, queues and so on), AMQP 0-9-1 includes the concept of virtual hosts (vhosts). They are similar to virtual hosts used by many popular Web servers and provide completely isolated environments in which AMQP entities live. Protocol clients specify what vhosts they want to use during connection negotiation.

Ngoài ra còn có các khái niệm khác sẽ được giải chi tiết ở phần sau.

Để cho dễ hình dung thì chúng ta sẽ chia quá trình gửi một Message từ producer tới Consume thành 3 giai đoạn: publishing, routing và consuming

## **1. publishing**

Đây là quá trình chương trình gửi message và hệ thống message broker.

Message Attributes and Payload

Messages in the AMQP 0-9-1 model have attributes. Some attributes are so common that the AMQP 0-9-1 specification defines them and application developers do not have to think about the exact attribute name. Some examples are

Content type

Content encoding

Routing key

Delivery mode (persistent or not)

Message priority

Message publishing timestamp

Expiration period

Publisher application id

Some attributes are used by AMQP brokers, but most are open to interpretation by applications that receive them. Some attributes are optional and known as headers. They are similar to X-Headers in HTTP. Message attributes are set when a message is published.

Messages also have a payload (the data that they carry), which AMQP brokers treat as an opaque byte array. The broker will not inspect or modify the payload. It is possible for messages to contain only attributes and no payload. It is common to use serialisation formats like JSON, Thrift, Protocol Buffers and MessagePack to serialize structured data in order to publish it as the message payload. Protocol peers typically use the “content-type” and “content-encoding” fields to communicate this information, but this is by convention only.

Messages may be published as persistent, which makes the broker persist them to disk. If the server is restarted the system ensures that received persistent messages are not lost. Simply publishing a message to a durable exchange or the fact that the queue(s) it is routed to are durable doesn’t make a message persistent: it all depends on persistence mode of the message itself. Publishing messages as persistent affects performance (just like with data stores, durability comes at a certain cost in performance).

Learn more in the Publishers guide.

Đây là giai đoạn Producer gửi message vào Exchange. Có 4 loại exchange Direct exchange, Fanout exchange, Topic exchange , Headers exchange. Tất cả các loại exchange trên đều bao gồm các thuộc tính quan trọng name, Durability, Auto-delete, Arguments.

## **2. Routing**

Đây là giai đoạn xảy ra trong nội tại rabbitmq. Message sẽ được chuyển từ Exchange đến Queue thông qua routing key và header.

Queue chứa message cái mà sẽ được comsume bởi Consumer. ngoài ra một queue phải bao gồm các thuộc tính: Name, Durable, Exclusive, Auto-delete, Arguments.

Note:

- Để sử dụng 1 queue, thì nó phải được khai báo. Nếu queue chưa tồn tại, nó sẽ được tạo. Nhưng nếu queue đã tồn tại thì việc khai báo sẽ không có tác dụng, nhưng nếu việc khai báo không trùng với thông tin queue đã tạo thì exception with code 406 (PRECONDITION_FAILED) will be raised.
- Queue name không thể trùng với pattern "amq."Bindings are rules that exchanges use (among other things) to route messages to queues. To instruct an exchange E to route messages to a queue Q, Q has to be bound to E. Bindings may have an optional routing key attribute used by some exchange types. The purpose of the routing key is to select certain messages published to an exchange to be routed to the bound queue. In other words, the routing key acts like a filter.

To draw an analogy:

Queue is like your destination in New York city

Exchange is like JFK airport

Bindings are routes from JFK to your destination. There can be zero or many ways to reach it

Having this layer of indirection enables routing scenarios that are impossible or very hard to implement using publishing directly to queues and also eliminates certain amount of duplicated work application developers have to do.

If a message cannot be routed to any queue (for example, because there are no bindings for the exchange it was published to) it is either dropped or returned to the publisher, depending on message attributes the publisher has set.

## **3. Receiving**

Storing messages in queues is useless unless applications can consume them. In the AMQP 0-9-1 Model, there are two ways for applications to do this:

Subscribe to have messages delivered to them (“push API”): this is the recommended option

Polling (“pull API”): this way is highly inefficient and should be avoided in most cases

With the “push API”, applications have to indicate interest in consuming messages from a particular queue. When they do so, we say that they register a consumer or, simply put, subscribe to a queue. It is possible to have more than one consumer per queue or to register an exclusive consumer (excludes all other consumers from the queue while it is consuming).

Each consumer (subscription) has an identifier called a consumer tag. It can be used to unsubscribe from messages. Consumer tags are just strings.

Consumers

Storing messages in queues is useless unless applications can consume them. In the AMQP 0-9-1 Model, there are two ways for applications to do this:

Subscribe to have messages delivered to them (“push API”): this is the recommended option

Polling (“pull API”): this way is highly inefficient and should be avoided in most cases

With the “push API”, applications have to indicate interest in consuming messages from a particular queue. When they do so, we say that they register a consumer or, simply put, subscribe to a queue. It is possible to have more than one consumer per queue or to register an exclusive consumer (excludes all other consumers from the queue while it is consuming).

Each consumer (subscription) has an identifier called a consumer tag. It can be used to unsubscribe from messages. Consumer tags are just strings.

Message Acknowledgements

Consumer applications – that is, applications that receive and process messages – may occasionally fail to process individual messages or will sometimes just crash. There is also the possibility of network issues causing problems. This raises a question: when should the broker remove messages from queues? The AMQP 0-9-1 specification gives consumers control over this. There are two acknowledgement modes:

After broker sends a message to an application (using either basic.deliver or basic.get-ok method).

After the application sends back an acknowledgement (using the basic.ack method).

The former choice is called the automatic acknowledgement model, while the latter is called the explicit acknowledgement model. With the explicit model the application chooses when it is time to send an acknowledgement. It can be right after receiving a message, or after persisting it to a data store before processing, or after fully processing the message (for example, successfully fetching a Web page, processing and storing it into some persistent data store).

If a consumer dies without sending an acknowledgement, the broker will redeliver it to another consumer or, if none are available at the time, the broker will wait until at least one consumer is registered for the same queue before attempting redelivery.

Rejecting Messages

When a consumer application receives a message, processing of that message may or may not succeed. An application can indicate to the broker that message processing has failed (or cannot be accomplished at the time) by rejecting a message. When rejecting a message, an application can ask the broker to discard or requeue it. When there is only one consumer on a queue, make sure you do not create infinite message delivery loops by rejecting and requeueing a message from the same consumer over and over again.

Negative Acknowledgements

Messages are rejected with the basic.reject method. There is one limitation that basic.reject has: there is no way to reject multiple messages as you can do with acknowledgements. However, if you are using RabbitMQ, then there is a solution. RabbitMQ provides an AMQP 0-9-1 extension known as negative acknowledgements or nacks. For more information, please refer to the Confirmations and basic.nack extension guides.

Prefetching Messages

For cases when multiple consumers share a queue, it is useful to be able to specify how many messages each consumer can be sent at once before sending the next acknowledgement. This can be used as a simple load balancing technique or to improve throughput if messages tend to be published in batches. For example, if a producing application sends messages every minute because of the nature of the work it is doing.

Note that RabbitMQ only supports channel-level prefetch-count, not connection or size based prefetching.

## **Exchange type Default pre-declared names**

Direct exchange (Empty string) and amq.direct

Fanout exchange amq.fanout

Topic exchange amq.topic

Headers exchange amq.match (and amq.headers in RabbitMQ)

Besides the exchange type, exchanges are declared with a number of attributes, the most important of which are:

Name

Durability (exchanges survive broker restart)

Auto-delete (exchange is deleted when last queue is unbound from it)

Arguments (optional, used by plugins and broker-specific features)

## **Queue**

Name

Durable (the queue will survive a broker restart)

Exclusive (used by only one connection and the queue will be deleted when that connection closes)

Auto-delete (queue that has had at least one consumer is deleted when last consumer unsubscribes)

Arguments (optional; used by plugins and broker-specific features such as message TTL, queue length limit, etc)

Before a queue can be used it has to be declared. Declaring a queue will cause it to be created if it does not already exist. The declaration will have no effect if the queue does already exist and its attributes are the same as those in the declaration. When the existing queue attributes are not the same as those in the declaration a channel-level exception with code 406 (PRECONDITION_FAILED) will be raised.

## **Queue Names**

Applications may pick queue names or ask the broker to generate a name for them. Queue names may be up to 255 bytes of UTF-8 characters. An AMQP 0-9-1 broker can generate a unique queue name on behalf of an app. To use this feature, pass an empty string as the queue name argument. The generated name will be returned to the client with queue declaration response.

Queue names starting with “amq.” are reserved for internal use by the broker. Attempts to declare a queue with a name that violates this rule will result in a channel-level exception with reply code 403 (ACCESS_REFUSED).