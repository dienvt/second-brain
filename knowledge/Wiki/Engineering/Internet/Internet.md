---
title: "First thing first"
date: 2026-01-14
tags:
  - engineering
  - networking
---

First thing first

References

Protocol

TCP vs UDP

HTTP/HTTPS

DNS (Domain Name System)

gRPC

Why gRPC fast

HTTP2

# First thing first

The Internet is a large network of computers which communicate all together.

The various technologies that support the Internet have evolved over time, but the way it works hasn't changed that much: Internet is a way to connect computers all together and ensure that, whatever happens, they find a way to stay connected.

## References

[https://youtu.be/zN8YNNHcaZc](https://youtu.be/zN8YNNHcaZc)

# Protocol

**Overview of the TCP/IP Model**

1. **Application Layer (OSI: Application, Representation, Session)**
    - This topmost layer deals with user interactions, applications, and protocols.
    - It's where communication between applications occurs.
    - It manages high-level protocols such as HTTP, FTP, SMTP, DNS, and more.
    - Encodes and formats data for network presentation.

1. **Transport Layer (OSI: Transport)**
    - The Transport Layer is responsible for end-to-end communication between devices.
    - It ensures data integrity, reliability, and flow control.
    - It encompasses two primary protocols: TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).
    - TCP provides reliable, connection-oriented communication, while UDP offers faster, connectionless communication.

1. **Internet Layer (OSI: Network)**
    - This layer handles the addressing, routing, and packaging of data for transmission across networks.
    - It uses IP (Internet Protocol) addressing to uniquely identify devices on a network.
    - IP packets contain both source and destination addresses, enabling routers to forward data across multiple networks.

1. **Network Layer (OSI: Data-link, Physical)**
    - The Link Layer, also known as the Network Interface Layer, deals with physical connections and local networks.
    - It manages communication between devices on the same network segment.
    - Handles protocols like Ethernet, Wi-Fi (802.11), and PPP (Point-to-Point Protocol).
    - Encapsulates IP packets into frames and manages data transmission over physical media.

## TCP vs UDP

TCP (Transmission Control Protocol) and UDP (User Datagram Protocol) are both protocols used in network communication, but they have significant differences in terms of their functionalities and the way they handle data transmission:

1. **Connection Orientation:**
    - TCP is connection-oriented, meaning it establishes a connection between the sender and receiver before transmitting data. It ensures reliable and ordered delivery of packets.
    - UDP is connectionless; it does not establish a connection before sending data. It's a fire-and-forget protocol that doesn't guarantee delivery or order of packets.
2. **Reliability:**
    - TCP ensures reliability by using acknowledgments, retransmissions, and sequencing of packets. If a packet is lost, it will be retransmitted. This makes TCP suitable for applications that require guaranteed delivery, such as web browsing, email, and file transfer.
    - UDP does not have built-in mechanisms for error checking, acknowledgment, or packet ordering. Therefore, it's faster but less reliable. It's suitable for real-time applications like video streaming, online gaming, and VoIP where a small amount of packet loss is acceptable.
3. **Packet Structure:**
    - TCP adds additional header information for sequencing, acknowledgment, flow control, and error checking. This makes TCP packets larger and more resource-intensive.
    - UDP has a simpler header structure, resulting in smaller packet sizes and lower overhead. This simplicity makes UDP faster but less robust.
4. **Order of Delivery:**
    - TCP ensures that data packets arrive in the same order they were sent. If a packet arrives out of order, TCP will rearrange them before passing them to the application.
    - UDP does not guarantee packet order. If packets are sent in a certain order, there's no guarantee they will arrive in the same order.
5. **Usage:**
    - TCP is suitable for applications that require reliability and accuracy in data delivery, where retransmission of lost packets is acceptable at the expense of some overhead. It's commonly used for HTTP, FTP, SSH, etc.
    - UDP is used for applications where speed and lower latency are more critical than guaranteed delivery. Real-time communication applications like video streaming, online gaming, DNS, and VoIP often use UDP.

In summary, TCP provides reliable, ordered, and error-checked delivery at the cost of higher overhead, while UDP sacrifices these features for speed and lower latency, making it suitable for time-sensitive applications where occasional packet loss is acceptable. The choice between TCP and UDP depends on the specific requirements and priorities of the application or service being used.

## HTTP/HTTPS

**HTTP (Hypertext Transfer Protocol):**

- HTTP is the standard protocol used for transferring hypertext (text, images, videos, etc.) over the web.
- It operates on top of TCP/IP and is considered a stateless protocol, meaning each request-response cycle is independent of previous ones.
- It transmits data in plain text, making it susceptible to eavesdropping and tampering by malicious actors.
- Information sent via HTTP is not encrypted, so sensitive data like passwords, credit card details, and personal information can be intercepted and read easily by attackers.

**HTTPS (Hypertext Transfer Protocol Secure):**

- HTTPS is an extension of HTTP that uses encryption protocols (most commonly SSL/TLS) to secure the data being transmitted.
- It adds a layer of security by encrypting the data transferred between the client (e.g., a web browser) and the server.
- Encryption ensures that even if someone intercepts the data, it appears as an unreadable string of characters, making it extremely difficult to decipher without the encryption key.
- It provides authentication, ensuring that users are communicating with the intended website and not an impostor or a middleman trying to intercept the data.

The key difference between HTTP and HTTPS lies in the security aspect: HTTPS encrypts data during transmission, offering a secure way to transmit sensitive information over the internet, while HTTP does not provide encryption, leaving data vulnerable to interception and manipulation.

  

# DNS (Domain Name System)

[https://youtu.be/Wj0od2ag5sk?feature=shared](https://youtu.be/Wj0od2ag5sk?feature=shared)

[https://howdns.works](https://howdns.works/)

TLD: **Top-Level Domain** server

![[dns_record_request_sequence_recursive_resolver.png]]

  

# gRPC

## Why gRPC fast

**Is gRPC fast:**

[https://medium.com/@LadyNoBug/grpc-v-s-rest-v-s-others-5d8b6eaa61df](https://medium.com/@LadyNoBug/grpc-v-s-rest-v-s-others-5d8b6eaa61df)

[https://learn.microsoft.com/en-us/aspnet/core/grpc/performance?view=aspnetcore-8.0](https://learn.microsoft.com/en-us/aspnet/core/grpc/performance?view=aspnetcore-8.0)

[https://stackoverflow.com/questions/45625886/rest-vs-grpc-when-should-i-choose-one-over-the-other](https://stackoverflow.com/questions/45625886/rest-vs-grpc-when-should-i-choose-one-over-the-other)

gRPC is fast due to several reasons:

1. **Protocol Buffers (Protobuf):** gRPC uses Protobuf as its default serialization mechanism. Protobuf is a binary serialization format that is highly efficient in terms of both processing speed and message size. Its binary nature reduces the amount of data sent over the network, making communication faster.
2. **HTTP/2 Protocol:** gRPC utilizes HTTP/2 as its underlying protocol. HTTP/2 introduces features like multiplexing, header compression, and server push, which enhance the efficiency of communication between the client and the server. Multiplexing allows multiple requests to be sent concurrently over a single TCP connection, reducing latency and improving throughput.
3. **Binary Data Exchange:** gRPC exchanges data in a binary format, which is more compact and faster to parse compared to text-based formats like JSON or XML. This results in quicker serialization and deserialization processes, contributing to overall speed improvements.
4. **Streaming Support:** gRPC offers both unary RPC (request/response) and streaming capabilities. Streaming allows for continuous communication between the client and server, enabling efficient handling of large data sets or real-time updates without the need for repeated connections, thereby enhancing performance.
5. **Code Generation:** gRPC generates client and server code using Protocol Buffers. This generated code is highly optimized for performance, providing a streamlined interface for communication between services.

Combining these factors results in gRPC being a fast and efficient framework for building distributed systems, especially in scenarios where high performance and low latency are crucial, such as microservices architectures and communication between backend services in large-scale applications

  

# HTTP2

references:

[https://200lab.io/blog/http2-la-gi/](https://200lab.io/blog/http2-la-gi/)

  

1. **HTTP/1.1 Head-of-Line Blocking:** When multiple requests are made over the same connection, if one request is slow to resolve (due to factors like network latency or server processing), it blocks subsequent requests from being processed. All other requests queued behind the slow one must wait for it to complete, leading to inefficient resource utilization and increased page load times.