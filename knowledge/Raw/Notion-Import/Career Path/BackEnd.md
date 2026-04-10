[https://roadmap.sh/backend](https://roadmap.sh/backend)

  

Concept

OOP

AOP

FP

S.O.L.I.D

A.C.I.D properties

C.A.P theorem

Architecture

Stateless vs Stateful

Service Mesh

Interview

What is software design

Why Design and architecture is important?

What is Software Architecture do?

How to express Software

What is the principle do you follow when design software?

Career, Skill and Advices

When you interested

Typical career path

Exciting thing when you be a SA

Skills

How to staying up to date

Advices

Software requirements, Conceptual Design and Technical Design

Entity Objects

Boundary Objects

Control Objects

Categories of Objects in Action

Distributed Transaction

2PC

Introduction

Limitation

Saga

Introduction

Choreography

Orchestration

References

# Concept

**Cohesion:** Cohesion refers to the degree to which the elements (functions, methods, variables) within a module or class are related and work together to achieve a common purpose or responsibility. It measures the strength of the interconnections among the elements within an entity. **High cohesion** indicates that the elements within a module or class are closely related and focused on a single, well-defined purpose. In a **highly cohesive** module or class, the elements share common functionality, operate on the same set of data, and collaborate closely to achieve a specific goal. On the other hand, **low cohesion** suggests that the elements within a module or class are loosely related or have unrelated responsibilities. Low cohesion may lead to code that is harder to understand, maintain, and modify, as the responsibilities of the module or class are scattered or not well-defined.

**Coupling:** Coupling refers to the degree of interdependence or connectivity between different modules, classes, or components within a software system. It measures how closely one entity relies on another entity. **Low coupling** indicates loose or weak dependencies between entities. **High coupling** means that there are strong dependencies between entities, where changes in one entity can have a significant impact on other entities

  

## OOP

**“Object-orient­ed program­ming is a par­a­digm based on the con­cept of wrap­ping pieces of data, and behav­ior relat­ed to that data, into spe­cial bun­dles called objects, which are con­struct­ed from a set of “blue­prints”, defined by a pro­gram­mer, called class­es.”**

**Abstraction**: Abstraction involves capturing the essential features and behavior of an object or a class, while hiding unnecessary details. It focuses on defining interfaces and functionality without getting into implementation specifics. Abstraction helps manage complexity and enables modular design.

**Encapsulation**: Encapsulation is the process of bundling data and related methods into a class, hiding the internal implementation details from the outside. It provides data protection and helps ensure that changes to the internal state of an object are done through well-defined interfaces.

**Polymorphism**: Polymorphism allows objects of different classes to be treated as objects of a common superclass. It enables the same method to be executed differently based on the type of the object. Polymorphism promotes flexibility and modularity in code design.

**Inheritance**: Inheritance allows classes to inherit properties and methods from other classes, forming a hierarchical relationship. Subclasses (derived classes) can inherit and extend the characteristics of a superclass (base class). Inheritance promotes code reuse, extensibility, and the concept of "is-a" relationships.

  

### Relationship between objects

**Association:** Associate between 2 objects that completely separate.

**Aggregation**: a weak “Has-a” relationship. Object А knows about object B, and con­sists of B. Class A depends on B. But when one in each object be destroyed, the another still exist.

**Com­po­si­tion**: a strong “Has-a” relationship. Object А knows about object B, con­sists of B, and man­ages B’s life cycle. Class A depends on B.

> Depending on your design, you can relate wholes to parts in different increasingly tighter ways

  

Imple­men­ta­tion: Class А defines meth­ods declared in inter­face B. Objects A can be treat­ed as B. Class A depends on B.

Inher­i­tance: Class А inher­its inter­face and imple­men­ta­tion of class B but can extend it. Objects A can be treat­ed as B. Class A depends on B.

  

## AOP

Key Concepts of Aspect-Oriented Programming (AOP):

1. **Aspect:**  
    An aspect is a modular unit of cross-cutting functionality. It encapsulates a concern, such as logging or security, and defines how that concern should be applied to different parts of the program.  
    
2. **Join Point:**  
    A join point is a specific point in the execution of a program, such as a method call, object instantiation, or field access. Aspects can be applied at specific join points to modify the behavior of the program.  
    
3. **Advice:**  
    Advice is the actual code that gets executed at a particular join point. It defines what should happen before, after, or around a join point. Examples include "before" advice that performs actions before a method call, and "after" advice that performs actions after a method call.  
    
4. **Pointcut:**  
    A pointcut defines a set of join points where advice should be applied. It specifies the criteria for selecting join points based on method signatures, class names, and other contextual information.  
    
5. **Weaving:**  
    Weaving is the process of integrating aspects into the main codebase. It can happen at different times: compile-time, load-time, or runtime. During weaving, the advice defined in aspects is woven into the appropriate join points.  
    

## FP

Key Concepts of Functional Programming:

1. **Pure Functions:**  
    Pure functions are the cornerstone of functional programming. They always produce the same output for the same input and do not have side effects (modifying external state or data). This makes them predictable and easier to reason about.  
    
2. **Immutability:**  
    In FP, data is treated as immutable, meaning it cannot be changed once created. Instead of modifying existing data, new data is created. Immutability simplifies concurrency and helps prevent unintended side effects.  
    
3. **First-Class and Higher-Order Functions:**  
    Functions in FP are first-class citizens, which means they can be passed as arguments to other functions, returned from functions, and stored in variables. Higher-order functions are functions that take other functions as arguments or return them as results.  
    
4. **Function Composition:**  
    Functional programming encourages combining small, focused functions to create more complex functions. This is known as function composition and helps create modular and reusable code.  
    
5. **Recursion:**  
    Recursion is a common technique in functional programming for solving problems by breaking them down into smaller instances of the same problem. It replaces iterative loops and can lead to elegant solutions.  
    
6. **Referential Transparency:**  
    An expression is said to be referentially transparent if it can be replaced with its value without affecting the program's behavior. This property allows for easier reasoning about code and optimization.  
    
7. **Lazy Evaluation:**  
    Lazy evaluation is a strategy where expressions are not evaluated until their values are actually needed. This can lead to more efficient use of resources, especially when dealing with infinite data streams.  
    
8. **Pattern Matching:**  
    Pattern matching is a way to destructure and match complex data structures in a concise and readable manner. It's commonly used in functional programming languages to handle different cases of data.  
    

## S.O.L.I.D

Single Responsibility Principle (SRP): A class should have only one reason to change, meaning it should have a single responsibility or purpose. This principle promotes high cohesion by ensuring that a class is focused on a single task.

  

Open/Closed Principle (OCP): Software entities (classes, modules, functions, etc.) should be open for extension but closed for modification. In other words, the behavior of a class should be extendable without modifying its existing code.

  

Liskov Substitution Principle (LSP): Objects of a superclass should be replaceable with objects of its subclasses without breaking the integrity of the program. Subtypes should be substitutable for their base types.

  
Interface Segregation Principle (ISP): Clients should not be forced to depend on interfaces they do not use. This principle encourages the creation of smaller and more specific interfaces instead of large, monolithic ones.  

  
Dependency Inversion Principle (DIP): High-level modules should not depend on low-level modules; both should depend on abstractions. This principle promotes loose coupling between modules and allows for easier changes in the dependencies without affecting the overall system.  

## A.C.I.D properties

1. Atomicity: Atomicity ensures that a transaction is treated as a single, indivisible unit of work. It means that either all the operations within a transaction are successfully completed, or none of them are applied to the database. If any part of a transaction fails, all changes made by the transaction are rolled back, and the database remains unchanged.
2. Consistency: Consistency ensures that a transaction brings the database from one valid state to another valid state. The database must satisfy certain integrity constraints defined by the database schema, ensuring that data is valid and follows predefined rules. Consistency guarantees that the database is in a consistent state before and after a transaction.
3. Isolation: Isolation ensures that concurrent execution of transactions does not interfere with each other. Each transaction should appear to be executing in isolation, as if it were the only transaction being executed. Isolation prevents issues like dirty reads, non-repeatable reads, and phantom reads, maintaining data integrity and preventing conflicts between concurrent transactions.
4. Durability: Durability guarantees that once a transaction is committed, its changes are permanent and will survive any subsequent failures, such as power outages or system crashes. The changes made by a committed transaction are stored in a durable storage medium (such as disk) and can be recovered in case of a failure.

## C.A.P theorem

The CAP theorem, also known as Brewer's theorem, is a fundamental principle in distributed computing that states that it is impossible for a distributed computer system to simultaneously provide all three of the following guarantees:

1. Consistency: Every read operation in the system receives the most recent write or an error. Consistency ensures that all nodes in a distributed system see the same data at the same time.
2. Availability: Every request made to the system receives a response, without any guarantee of the data being the most recent. Availability ensures that the system remains operational and responsive even in the face of failures.
3. Partition tolerance: The system continues to operate even when network partitions (communication failures) occur, resulting in the loss of message delivery between some nodes. Partition tolerance ensures that the system can handle and recover from network failures.

According to the CAP theorem, in the presence of a network partition (P), a distributed system must choose between either consistency (C) or availability (A). In other words, when a partition occurs, a system can either guarantee consistency and sacrifice availability (CP), or guarantee availability and sacrifice consistency (AP). It is not possible to have both consistency and availability under such circumstances.

# Architecture

## Stateless vs Stateful

**Stateless**

In a stateless system, each request from a client to a server is considered independent and self-contained. The server does not retain any information or context about previous requests from the same client. Each request must contain all the necessary information needed to process it, without relying on any stored data or session information on the server-side.

Stateless systems are easier to scale and maintain since there is no need to manage and synchronize session data across multiple servers. HTTP is an example of a stateless protocol.

Stateless systems are simpler to design, implement, and maintain. Each request contains all the information needed to process it, reducing the complexity on the server side.

Since there is no session state to maintain, failures in one part of the system can be easily handled, and the requests can be routed to other healthy servers without worrying about losing session data.

Each request is self-contained and does not depend on previous interactions, which leads to more consistent behavior

**Statefull**

In a stateful system, the server keeps track of the state or context of each client's session. When a client interacts with the server, the server stores and remembers certain information related to that client, such as session data or user authentication credentials. This allows the server to maintain continuity across multiple requests from the same client and offer a more personalized and interactive experience.

However, managing session state can be more complex, especially when dealing with distributed systems or high scalability requirements.

## Service Mesh

Modern applications are typically architected as distributed collections of microservices, with each collection of microservices performing some discrete business function. A service mesh is a dedicated infrastructure layer that you can add to your applications. It allows you to transparently add capabilities like observability, traffic management, and security, without adding them to your own code. The term “service mesh” describes both the type of software you use to implement this pattern, and the security or network domain that is created when you use that software.

A service mesh is a networking infrastructure layer that provides a way to manage communication between services in a microservices architecture. As modern applications become more complex and distributed, managing the communication and interaction between these services can become challenging. This is where a service mesh comes in.

As the deployment of distributed services, such as in a Kubernetes-based system, grows in size and complexity, it can become harder to understand and manage. Its requirements can include discovery, load balancing, failure recovery, metrics, and monitoring. A service mesh also often addresses more complex operational requirements, like A/B testing, canary deployments, rate limiting, access control, encryption, and end-to-end authentication.

Service-to-service communication is what makes a distributed application possible. Routing this communication, both within and across application clusters, becomes increasingly complex as the number of services grow. Istio helps reduce this complexity while easing the strain on development teams.

Popular service mesh implementations include Istio, Linkerd, and Consul Connect.

  

## Interview

What does a career in software design and architecture look like? What is the difference between software design and software architecture? These are questions that we will explore in more detail throughout the specialization. However, let's take a quick look at them now. Like many roles in the software industry, the software designer or a software architect role can look very different from company to company. Characteristics like company size, the scope of the project, the experience of the development team, the organizational structure and the age of the company can all impact what these roles look like. In some companies, there may be a distinct role for a software designer or architect. In other companies, the design may be completed by a member or members of the development team. Typically, the software designer role would be responsible for outlining a software solution to a specific problem by designing the details of individual components and their responsibilities. A software architect role would be responsible for looking at the entire system and choosing appropriate frameworks, data storage, solutions and determining how components interact with each other. That brings us to the primary difference between software design and software architecture. In short, software design looks at the lower level aspects of a system, whereas software architecture tends to look at the bigger picture, the higher level aspects of a system. Think of this like designing a building. An architect focuses on the major structures and services, while an interior designer focuses on the smaller spaces within. Great software designers and architects are detail-oriented, forward thinkers. They need to be able to see the product at both the low and high levels. They need to be creative problem solvers in order to come up with a quality solution for the problem at hand. And they need to be able to express these ideas effectively with the product manager and the development team. Does that sound like something you'd be good at? Software design and architecture is essential to the software development process. Let's take a look at what people in the industry feel about this role.

### What is software design

Software design is the process of turning the wishes and requirements of a customer into working code that is stable and maintainable in the long run, and can be evolved and can become part of a larger system. That's software design. I like the or because I don't make a distinction between software architecture and software design. I think they're just the same problem at a different scale.

Way I like to think of it is that architecture is primarily, begins with understanding what's the business problem that the client needs to solve. Where business doesn't mean necessarily financial business, any business. Once you've realized that that's your primary task, which is to figure out what the client wants, then everything kind of falls in after that. Because If you understand the problem, then you can start to think about what, in your previous experience, as possible solutions, and then you start getting a idea of how your overall solution is going to look like. And that's where I kind of say, really architecture is the study of boxes and lines. Because your first description of what it is you're trying to do is simply a set of boxes with things inside them and lines expressing relationships.

  

### Why Design and architecture is important?

Design and architecture is important if you want to have a stable, long- lived system. Anybody can build a system that'll last a week or a month or a year, but if you want to build something that is the basis of other people's work and contribution over potentially a period of years or longer, in some cases, you need to put some thought into it. You need to have somebody whose job it is to look out for the long game and make sure that you are not making suboptimal short term decisions. Architecture is important because if you get it wrong, your project will fail. That's it. It's just that simple.

We know it in the building world, and we know it in the software world. Where we're using the term architecture to be this understanding of the relationship between the requirements of the user and the ability to build a system that will deliver those requirements. I think you can trace back most major software failures to bad architecture, where architecture is used in this general sense. One of the key challenges in software architecture is the tendency to have to trade off between speed and quality, If I boil it down, right? I think there's a tendency for the customer in the business to want their, their software, to want their results as soon as possible. And there's a tendency for the engineering team to want to build, the most robust, thoroughly designed, thoroughly implemented system possible. And so we trading off between these things all the time. And I think, it's that tension in that trade off is where you get really good software, you get good designs out of that, but it's a process you have to go through to go through that. So our biggest issue that we face is understanding the client's problem. What is it they really want to do? And in many cases the client actually doesn't know what they want to do either. They come in with only a partial understanding, a vague kind of sense that they could be doing things better. But often, one of our first task is to actually help them understand, with more precision, what their business is.

### What is Software Architecture do?

A software architect's job is to be the interface between the product and the customer and the engineering teams. And so for instance, customers will express a requirement or a need they have of the, of the software and it's the architect's job to then work with the customers and their representatives, product managers and such, to come up with the technical requirements of how we're going to solve the problem. And then they take those requirements to the engineering teams and worked with the engineers on how to realize that in a way that is meeting the customer's requirements and also aligned with the technical best practices and nonfunctional requirements that have to be adhered to in the product. The software architect is like a building architect. They're responsible for the overall conceptual integrity of the project. Their main goal is to serve the needs of the customer within the budget that the customer has.

### How to express Software

I would express software design or software architecture in a couple of different ways. For small things, for simple things, you'll sit with an engineer and you'll whiteboard something out and you'll come up with a design that way and you'll basically get them going. For larger initiatives, larger projects, you're typically writing fairly substantial design specification documents, where you're exploring all the different possible use cases, all the different possible flow variations and things of this kind, in addition to all of those critical functional and nonfunctional requirements, stability, maintainability, these kinds of things. So, in general, I would say that we communicate software architecture through the written word, through wikis, through white papers, these kinds of things, in addition to fairly detailed engineering design schematics, class diagrams, if necessary, big box diagrams, if it's just a simple high level architecture design. I've been programming for 45 years and one thing I've learned is that the only thing that's really there is the code and everything else that you talk about is views on the code. So, I like to express architecture or describe architecture as saying that, all the things that I'm going to do, the boxes and lines, the prose, the fancy diagrams of the diagrams and napkins, there are simply indexes into the code. That's how you find your way to the actual artifact that's actually doing what you want to do.

### What is the principle do you follow when design software?

I tend to apply simplicity first as my main principle, if I'm looking at how I'm approaching the problem. That's the filter that I try to use on it. And I often will find myself, there's a tendency in people to complicate things, to inject complexity because it's interesting. As an engineer, as a technical person, complexity is fun and interesting. And it's only when you stripped away all the unnecessary complexity that you realize you've got the core of a great solution to a problem. And so, I really try to do that and when I'm working with product teams and engineers alike, that simplicity principle really helps to cut through a lot of the confusion.

What's the most important principle? Simplicity. That's true and it's the engineering maxim to keep it simple. The reason for that is twofold. One is that if it's simple, you probably have a pretty good chance of getting it right or almost right. That's one part. The other thing is if it's simple, then you can explain it to someone simply, that communication of architecture is important because you're not going to be around forever. And you need to transfer your knowledge over to someone else. And if it's not simple, the knowledge transfer cost is higher because it's more complicated to explain and the chances of misunderstanding are much higher.

### Career, Skill and Advices

### When you interested

So, I became interested in software design by working as an engineer and being exposed to larger scale code bases, you know, progressively over the years. When I first started, of course, I worked on mostly very, very small things. And I was very interested in how software was put together and the design at, a micro design level. And then, as I proceeded in my career and I started to be exposed to some fairly large pieces of software that served millions of people, I got really interested in how those things are put together and what is it that makes that successful. And how do you make sure that you're not having to re-implement this thing over and over and over again. And I found that to be a really interesting side of the business that I really hadn't explored before. And so it turns out I really enjoyed that.

I think, if you start writing code to do things, to play around, you start asking yourself questions about: well, why is it that this code is really nice to work with and this code here is horrible? And you start asking yourself questions about design and the difference between design and architecture in the software business is, there really isn't any because our business is all self-similar. Issues that you ask about programs are the same issues you can ask about big systems. And I think in my case, I just became interested in this fundamental understanding of the issue of building software artifacts. And then it just naturally scales up, at some point you're building systems with a hundred thousand lines of code. And then, you suddenly realize on your project team that you don't have a million lines of code. And you're now a software architect because you have a million lines of code whereas before you were just a programmer you only had 10,000 lines of code.

  

### Typical career path

Most architects that I know started as software engineers. Usually as an intern or a new grad and they work basically overtime. They work on progressively larger and larger pieces of the software that they're responsible for. And what happens is, you start to see those engineers get to a level of comfort where they start to push outside of the code base that they're indirectly responsible for. And they start involving themselves in discussions around the larger impacts to the system of the work that's being done. And that just generally continues until all of a sudden they're actually working at a much higher level of abstraction. And then contributing at a very different level and so that's how you know you've got an architect on your hands. Yeah, there's not a career path into software architecture.

What it really is is, if you think of architects as having more responsibility than programmers, what it really is as a career path where you get more and more responsibility, that you do by demonstrating that you're actually good at building things. My experience has been that I didn't think I was a expert programmer until I had been out in the world for 10 years and I think that's consistent with many of my colleagues. Over that period, you start working on bigger and bigger systems and eventually someone trusts you with being the point person to put together a design for a much larger system than you'd ever done before. And then once you've done one of those and it hasn't been a total disaster, you get an even bigger system. So I guess, it's gradual building of your reputation is what makes you into a software architect.

### Exciting thing when you be a SA

I would say the most exciting thing about being a software architect is the satisfaction of seeing the final product put together and out there and being used by real people to good effect, right. Because you spend all this time early on in a project and you have to fight for your nonfunctional requirements and you have to fight for how this is all going to be put together. And then when it all comes together and you've done all that negotiation and it's out there in the hands of a customer and it's valuable, there's a real sense of pride with that. I think, additionally, you also get a lot of satisfaction and pride from making the right call in terms of long term viability of a code base and of a project. And so seeing somebody be able to come to your product, maybe years later, and make some very business critical contribution extension of something that you designed, without having to redesign it, is very satisfying. It tells you that you hit it on the right mark.

What's exciting in architecture? Well in general, you don't want too much excitement because that's usually associated with some sort of looming disaster. But what's interesting about software architecture, and that continues to make it interesting, is that someone always has a problem that's slightly different than all the problems you've seen before, which means that your previous solutions aren't necessarily going to work and you get to do something new. So, it's the novelty that makes up architecture interesting.

### Skills

An architect has to have a number of important skills, obviously, deep technical expertise is table stakes. You have to be a technical guru, I think, at a certain level. In addition to that, you need to be able to communicate with people at the level that they want to be communicated with. So if you're talking to a business person, they don't want to hear about your code. They want to hear about their business problems. And they want to hear how you're solving their business problems. If you're talking to an engineer, they want to know the business context but they need you to talk to them about code. And so, it's really important to have that ability to understand how the person you're talking to wants to be communicated with. So empathic communication, I would say, is really important. Additionally, some basic functional skills like a little bit of project planning and organizational skills, being able to keep a backlog of work so that you don't forget about things. Be able to juggle a lot of different competing concerns at the same time is also a very important skill.

The most important ones are what I would call, the soft people skills that you need in order to get people to tell you what their requirements are. This is very hard actually, especially in situations of uncertainty. Clients are very reluctant typically, to tell you the things that they're really bad at. They like to tell you all the things that they know how to do but they're reluctant to express where their understanding of a problem is incomplete or where their business processes just don't work right. And, if you don't identify those areas, you've actually encountered a big risk in your project, because those are the areas that are the problem. The well understood parts of a client's needs are not an issue. It's the parts that are fuzzy and not well understood. But of all the technical skills that you've got, you need this meta skill, which is to look at various technologies and ideas and decide, is that going to be useful to me or not in my particular problem I'm trying to solve? So as an architect, you have to know a lot about what's out there. But not in a tremendous amount of detail because a lot of the stuff that's out there, isn't going to be useful to you, at least immediately. By the time you might need it, it's probably gone through 10 releases anyway and isn't the same thing. So you have to have the skill of being able to quickly assess various technologies and fit them into your understanding of the discipline. So new language comes out, you so say, "Oh yeah, this is yet another procedural language with nothing much different than all these other ones." Or you might see something else and says, "Oh, that's interesting. I wonder if this particular style of approaching the problem, perhaps, aspect oriented programming, just to pull something of the air, will actually help me solve my problem in a better way or express my problem in a better way."

### How to staying up to date

Well, staying up to date is a bit of a trick. It's about exposing yourself to as much as you can in the outside world and inside your own company, as well. But in particular, you know, look at what the big companies are doing. What's Apple doing? What's Google doing? What's Amazon doing? And you read their blogs. You play with their software. You get an account on whichever tool you want to use and you start using those things. And you use that for inspiration. And just to see how others are approaching architecture in their systems, right? So there's a number of levels of inspiration there, I think. Additionally, read a lot of just the general tech press and find out what's going on out there in the world. Read academic journals for the appropriate areas and see what's coming a little farther down the line. What are the academics thinking about? So there's lots of those things to go after.

  

### Advices

So the advice I would give to a new software architect is to get as comfortable talking to people as you can and meet as many people as you as humanly possible. Expose yourself to as many ideas as you can. And share your own perspective as well. And I think it's by by leveraging the community, leveraging those around you, that you're going to be inspired to be creative in your architecture and you're going to get a better understanding of the context in which you're operating, both within your business as well as the broader technology landscape out there. And it'll help you to make better choices, ultimately.

The advice you give to new software architect is the same advice you give to a musician. Try and play with people who are much better than you are because that's how you become a better architect. And that means working with people who are better than you are. If you have the opportunity, at the very least, try to read as much of the foundational literature in the field and there's not that much to read. There's a maybe 20 key resources you should go to. Some of them dating back to the original papers in the 70s about coupling and cohesion. And then, of course, writing code. **So you need to work with people that are better than you are. Read a lot of code and read a lot of code and that's how you become a software architect. Oh and of course, learning from your mistakes is also quite valuable.**

  

## Software requirements, Conceptual Design and Technical Design

```Mermaid
flowchart LR
CD(Conceptual Design\n+Conceptuals\n+Connection between conceptuals\+Mockup design) ---> TD(Technical Design\n+entity objects\n+Boundary objects\n+Control objects)
```

  

Organizing your software into **entity objects, boundary objects, and control objects** will allow your code to be more flexible, reusable, and maintainable. Try to start using them today to see their power!

  

### Entity Objects

**Entity objects** are the most familiar, because they correspond to some real-world entity in the problem space. If you have an object representing a chair in your software, then this is an entity object. If you have an object representing a building or a customer, these are all entity objects. Generally, these objects will know attributes about themselves. They will also be able to modify themselves, and have some rules for how to do so.

When you are identifying objects to include in your software and breaking down those objects into smaller objects, you will initially get entity objects. The other categories of objects will come later, as you start to think about the technical design of the software.

### Boundary Objects

**Boundary objects** are objects which sit at the boundary between systems. This could be an object that deals with another software system - like an object that obtains information from the Internet. It could also be an object with the responsibility of showing information to the user and getting their input. If you program a user interface - the visual aspect of software - you are probably mostly working with boundary objects. Any object that deals with another system - a user, another software system, the Internet - can be considered a boundary object.

### Control Objects

**Control objects** are objects which are responsible for coordination. You will discover control objects when you attempt to break down a large object, and find that it would be useful to have an object that controls the other objects. You will see many examples of these objects in real usage in the next course in the specialization: Design Patterns. A great example is a Mediator: it simply coordinates the activities of many different objects so that they can stay loosely coupled.

### Categories of Objects in Action

At this point it may be difficult to see how these object types can help you. That is okay; breaking down objects in the best way takes real-world practice and experience. The most important thing to realize at this point is that your software will not solely consist of **entity objects**. Of course there will be objects for real-world items like tables and chairs or invoices and shopping carts, but there must also be objects for coordination and for interfacing with outside systems. They are a little bit harder to see, but no less essential, especially as you move from small projects to more complex software.

Organizing your software into **entity objects, boundary objects, and control objects** will allow your code to be more flexible, reusable, and maintainable. Try to start using them today to see their power!

# Distributed Transaction

## 2PC

### Introduction

2PC stand for "Two-Phase Commit" or “Prepare” and “Commit”. Two-Phase Commit is a distributed transaction protocol used in computer science and database systems to ensure that multiple databases or resources are updated consistently and in a coordinated manner.

The protocol works in two phases:

1. **Voting Phase (Phase 1):** In this phase, the coordinator (usually the server initiating the transaction) sends a "prepare to commit" message to all the participants (databases or nodes). Each participant responds with an acknowledgment indicating whether it can successfully commit the transaction. If any participant cannot commit, they respond with a "abort" message.
2. **Commit Phase (Phase 2):** If all participants respond with a "prepare to commit" acknowledgment, the coordinator sends a "commit" message to all participants. Upon receiving this message, each participant performs the actual commit of the transaction. If any participant had responded with an "abort" message in the voting phase, the coordinator sends an "abort" message to all participants, and they roll back the transaction.

### Limitation

**Blocking Behavior:** In the 2PC protocol, if any participant (database or node) becomes unavailable or fails to respond during the commit phase, the entire protocol can enter a blocking state

  

## Saga

### Introduction

In the context of micro-services and distributed transactions, a saga is a design pattern used to manage long-lived and multi-step transactions in a decentralized manner

A saga breaks down a complex transaction into a series of smaller, manageable steps or sub-transactions. Each step of the saga is represented by a specific action, which is a unit of work performed by an individual service. **These actions are typically idempotent**, meaning they can be executed multiple times without changing the final result.

The key characteristic of a saga is that it acknowledges that failures can happen at any point during the transaction's execution. When a step in the saga fails, **compensating** actions are executed to revert the effects of the previous steps, effectively undoing the changes and maintaining data consistency.

Summary for example: A saga is a assemble of T1,T2,T3,…Tn or T1,T2,T3,…Tn, C1, C2,…Cn which T is the smallest steps and c is the compensate of these smallest step.

  
  
**Saga trade of Atomicity to achieve Availability**

### **Choreography**

Choreography is an approach where each service or component in a system is responsible for its own behavior and communication. Services collaborate with each other by exchanging events or messages. Each service knows how to react to events it receives, and there is no central controller orchestrating the flow of actions. This means that services are loosely coupled and can evolve independently, which promotes flexibility and scalability.

In a choreographed system, services act autonomously and publish events that other services can react to. It's like a dance where each dancer (service) knows their steps and movements, and they coordinate with others by following the rhythm of the music (events/messages). This approach is often associated with event-driven architectures.

In summary, **Choreography** is decentralized, where each service knows its role and collaborates with others through events/messages

**Choreography-based Saga** is an approach which the coordination of the saga steps is decentralized. Each service involved in the saga emits events to notify other services about the progress or completion of their actions. Other services react to these events and continue the saga by performing their own actions accordingly. This approach resembles the choreography pattern discussed earlier.

### **Orchestration**

Orchestration, on the other hand, involves a central entity called the orchestrator that controls the flow of actions and coordinates the interactions between different services or components. The orchestrator defines the sequence of steps or activities and decides which services should perform each step. It acts as a conductor, directing the execution of the entire process.

In an orchestrated system, the orchestrator is responsible for making high-level decisions and delegating tasks to individual services. Think of it as a conductor leading an orchestra, where each musician (service) follows the conductor's directions to play their part at the right time. This approach is often associated with workflow management systems and is useful when complex business processes need to be managed and controlled.

In summary, **Orchestration** is centralized, where a controlling entity (the orchestrator) directs the sequence of actions and interactions between services.

**Orchestration-based Saga** is approach which centralizes the coordination of the saga via **Saga Execution Coordinator (SEC)**. There is a dedicated orchestrator that determines the sequence of steps and communicates directly with the individual services to instruct them on what actions to perform. The orchestrator is responsible for driving the saga's progress and ensuring its successful completion.

1. Order Service saves a pending order and asks Saga Execution Coordinator (SEC) to start a create order transaction.
2. SEC sends an Execute Payment command to Payment Service, and it replies with a Payment Executed message.
3. SEC sends a Prepare Order command to Stock Service, and it replies with an Order Prepared message.
4. SEC sends a Deliver Order command to Delivery Service, and it replies with an Order Delivered message.

### References

[https://youtu.be/xDuwrtwYHu8](https://youtu.be/xDuwrtwYHu8)