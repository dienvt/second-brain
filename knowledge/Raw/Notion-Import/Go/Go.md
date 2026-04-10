Ref

[[Go Programming Language: Channels and Goroutines]]

[[Goroutines]]

[[Types]]

Goroutines & Channel

Context

GOMAXPROCS

Channels

Under the hood

Buffered Channels

Slice

NOTE

[]byte vs string

GoGC

Mark Setup

Marking (concurent)

Mark assist

Mark termination

Sweeping

How does runtime know when to start a collection?

Defer

Memories

Reflection capabilities

Conclusion

# Ref

[https://roadmap.sh/golang](https://roadmap.sh/golang)

[[swag-go]]

[[Useful libs]]

[[GORM]]

  

# Go Programming Language: Channels and Goroutines

  

What is process, thread

[https://go.dev/blog/pipelines](https://go.dev/blog/pipelines)

[https://www.youtube.com/watch?v=4rLW7zg21gI](https://www.youtube.com/watch?v=4rLW7zg21gI)

  

In addition to powerful data structures, Go also has powerful concurrency features that make it easy to write concurrent programs. In this document, we will discuss two key features of Go's concurrency model: channels and goroutines.

## Goroutines

A goroutine is a lightweight thread of execution that is managed by Go's runtime. Goroutines are similar to threads, but they are cheaper to create and manage. You can create a goroutine using the `go` keyword followed by a function call. For example, here's how you would create a goroutine that prints "Hello, World!" in the background:

```Go
func main() {
    go fmt.Println("Hello, World!")
}
```

When you run this program, it will print "Hello, World!" in the background, concurrently with the main program. Since goroutines are lightweight, you can create many of them without using a lot of system resources.

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

[https://youtu.be/YHRO5WQGh0k?feature=shared](https://youtu.be/YHRO5WQGh0k?feature=shared)

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

  

  

  

## Channels

[https://youtu.be/KBZlN0izeiY?feature=shared](https://youtu.be/KBZlN0izeiY?feature=shared)

**Usage**

A channel is a typed conduit through which you can send and receive values between goroutines. Channels are used to communicate data between goroutines and synchronize their execution. Channels can be declared using the `make` function with the `chan` keyword followed by the type of the values that the channel will transmit. For example, here's how you would declare a channel that transmits integers:

```Plain
ch := make(chan int)
```

You can send a value to a channel using the `<-` operator. For example, here's how you would send the value `42` to a channel `ch`:

```Plain
ch <- 42
```

You can receive a value from a channel using the `<-` operator as well. For example, here's how you would receive the value sent to `ch` in the previous example:

```Plain
x := <-ch
```

If there is no value available on the channel, the receive operation will block the goroutine until a value is available.

  

### **Under the hood**

Channel struct is contain: **buf**(storage message), **lock**(lock write/read offset), **readx/writex**(index read/write), **sendq/recvq** (Send Queue and Receiver Queue when buf is full)

**sendq:** Sender store as a `sudog` contain the G (goroutine) which is waiting for send message to channel and element to send/recv

## Buffered Channels

By default, channels are unbuffered, which means that sending a value to a channel will block the sending goroutine until another goroutine receives the value. However, you can also create buffered channels, which have a fixed capacity and allow sending and receiving values to proceed without blocking until the channel is full. To create a buffered channel, you can specify the capacity of the channel when you declare it. For example, here's how you would create a buffered channel with a capacity of 3 that transmits integers:

```Plain
ch := make(chan int, 3)
```

You can send up to 3 values to this channel without blocking, but sending a fourth value will block the sending goroutine until another goroutine receives a value from the channel.

## Slice

```Go
var s []int
fmt.Printf("s len: %d cap: %d val: %v\n", len(s), cap(s), s)
//s len: 0 cap: 0 val: []
s = make([]int, 10)
fmt.Printf("s len: %d cap: %d val: %v\n", len(s), cap(s), s)
//s len: 10 cap: 10 val: [0 0 0 0 0 0 0 0 0 0]
s = make([]int, 0, 10)
fmt.Printf("s len: %d cap: %d val: %v\n", len(s), cap(s), s)
//s len: 0 cap: 10 val: []
s = make([]int, 10, 0) // error
fmt.Printf("s len: %d cap: %d val: %v\n", len(s), cap(s), s)
```

### **NOTE**

Must init array if know the capacity. Because when a slice pass through a function that can be replace without our aware

  

## []byte vs string

- **Mutability:** Strings are immutable, while byte slices are mutable. You can modify the elements of a byte slice directly, but you cannot modify individual characters in a string.
- **Data Representation:** Strings represent textual data encoded in UTF-8, while byte slices represent raw byte data.
- **Operations:** Strings have specific string manipulation functions and methods (e.g., `**strings**` package), while byte slices have methods and functions for byte manipulation (e.g., `**bytes**` package).
- **Use Cases:** Strings are suitable for representing text and working with human-readable data, while byte slices are used for handling binary data, byte streams, or non-UTF-8 encoded text.

  

# GoGC

references:

[https://tip.golang.org/doc/gc-guide](https://tip.golang.org/doc/gc-guide)

[https://tip.golang.org/doc/gc-guide#Eliminating_heap_allocations](https://tip.golang.org/doc/gc-guide#Eliminating_heap_allocations)

[https://www.golinuxcloud.com/golang-garbage-collector/#Tri-Color_Marking](https://www.golinuxcloud.com/golang-garbage-collector/#Tri-Color_Marking)

  

GoGC is none-moving, mark & sweep plan garbage collector

Step:

## Mark Setup

When a collection starts, the first activity that must be performed is turning on the [Write Barrier](https://en.wikipedia.org/wiki/Write_barrier), it allows the collector to maintain data integrity on the heap during a collection since both the collector and application goroutines will be running concurrently. To turn the Write Barrier on, every application goroutine running must be stopped. This activity is usually very quick, within 10 to 30 microseconds on average. _That is, as long as the application goroutines are behaving properly._

Suppose four goroutines are running before the GC is about to kick in. Each of these 4 goroutines must be stopped for GC for its work. The only way to do that is for the collector to watch and wait for each goroutine to make a function call. Function calls guarantee the goroutines are at a safe point to be stopped. What happens if one of those goroutines doesn’t make a function call (say it is performing a [tight loop operation](https://stackoverflow.com/a/2213001)), then what will happen?

For example, the 4th goroutine was performing the below code

`func stubbornGoroutine(numbers []int32) int { var r int32 for _, v := range numbers { // some operation to r } return r }`

This scenario could stall a garbage collection from starting. Since other processors can’t service any other goroutines while the collector waits. So, goroutines must make function calls in reasonable timeframes.

> A goroutine without function calls will not be preempted, and its P will not be released before the end of the task. That will force the “Stop the World” to wait for it.

## Marking (concurent)

Once the Write Barrier is turned on, the collector commences with the Marking phase.

The first thing the collector does is take **25%** of the available CPU capacity for itself. The collector uses Goroutines to do the collection work and needs the same P’s and M’s the application Goroutines use.

The marking phase consists of marking values in heap memory that are still in-use. This work starts by inspecting the stacks for all existing goroutines to find root pointers to heap memory. Then the collector must traverse the heap memory graph from those root pointers.

### **Mark assist**

If the collector determines that it needs to slow down allocations, it will recruit the application Goroutines to assist with the Marking work. This is called a **Mark Assist**. The amount of time any application Goroutine will be placed in a Mark Assist is proportional to the amount of data it’s adding to heap memory.

> Mark Assist helps finish the collection faster.

One goal of the collector is to eliminate the need for Mark Assists. If any given collection ends up requiring a lot of Mark Assist, the collector can start the next garbage collection earlier. This is done in an attempt to reduce the amount of Mark Assist that will be necessary for the next collection.

## Mark termination

Once the Marking work is done, the next phase is Mark Termination. This is when the Write Barrier is turned off, various clean up tasks are performed, and the next collection goal is calculated. Goroutines that find themselves in a tight loop during the Marking phase can also cause Mark Termination STW latencies to be extended.

Once the collection is finished, every P can be used by the application Goroutines again and the application is back to full throttle.

## Sweeping

Another activity happens after a collection is finished called Sweeping. Sweeping is when the memory associated with values in heap memory that were not marked as in-use are reclaimed. This activity occurs when application Goroutines attempt to allocate new values in heap memory. The latency of Sweeping is added to the cost of performing an allocation in heap memory and is not tied to any latencies associated with garbage collection.

## **How does runtime know when to start a collection?**

The collector has a **pacing algorithm** which determines when to start a collection. Pacing is modeled like a control problem where it is trying to find the right time to trigger a GC cycle so that it hits the target heap size goal. Go’s default pacer will try to trigger a GC cycle every time the heap size doubles. It does this by setting the next heap trigger size during the mark termination phase of the current GC cycle. So after marking all the live memory, it can make the decision to trigger the next GC when the total heap size is 2x what the live set currently is. The 2x value comes from a variable `GOGC` the runtime uses to set the trigger ratio.

One misconception is thinking that slowing down the pace of the collector is a way to improve performance. The idea being, if you can delay the start of the next collection, then you are delaying the latency it will inflict. Being sympathetic to the collector isn’t about slowing down the pace.

---

Go 1.5 was released in August 2015 with the new, low-pause, concurrent garbage collector, including an implementation of the [pacing algorithm](https://docs.google.com/document/d/1wmjrocXIWTr1JxU-3EQBI6BK6KgtiFArkG47XK73xIQ/edit#heading=h.4801yvqy4taz).

# Defer

The behavior of defer statements is straightforward and predictable. There are three simple rules:

1. _A deferred function’s arguments are evaluated when the defer statement is evaluated._
2. _Deferred function calls are executed in Last In First Out order after the surrounding function returns._
3. _Deferred functions may read and assign to the returning function’s named return values_

  

**Panic** is a built-in function that stops the ordinary flow of control and begins _panicking_. When the function F calls panic, execution of F stops, any deferred functions in F are executed normally, and then F returns to its caller. To the caller, F then behaves like a call to panic. The process continues up the stack until all functions in the current goroutine have returned, at which point the program crashes. Panics can be initiated by invoking panic directly. They can also be caused by runtime errors, such as out-of-bounds array accesses.

**Recover** is a built-in function that regains control of a panicking goroutine. Recover is only useful inside deferred functions. During normal execution, a call to recover will return nil and have no other effect. If the current goroutine is panicking, a call to recover will capture the value given to panic and resume normal execution.

  

**So if the library we using call panic, we just need to add recover to convert an panic to error and successfully handle it.**

# Memories

In Go (Golang), the term "escaping to the heap" refers to the situation where a variable that was originally allocated on the stack, which is a region of memory used for local variables and function call management, is promoted to the heap, which is a region of memory used for dynamically allocated data that persists beyond the scope of the current function.

Here's a more detailed explanation:

1. **Stack vs. Heap:** In Go, local variables, including function arguments and other temporary data, are typically allocated on the stack. The stack is a fast and efficient region of memory because it can be quickly allocated and deallocated as functions are called and return. However, the stack has limitations in terms of size and duration.
2. **Heap:** The heap, on the other hand, is a region of memory where data can persist for a longer duration. Data allocated on the heap is managed more explicitly, often by the programmer. Variables allocated on the heap can be accessed and modified from multiple parts of a program.
3. **Escape Analysis:** Go has a feature known as "escape analysis" that helps the compiler determine whether a variable can be allocated on the stack (in which case it's considered "not escaping") or whether it needs to be allocated on the heap (in which case it's considered "escaping").
    
    ```Bash
    go build -gcflags="-m"
    ```
    
4. **Reasons for Escaping:** Variables may escape to the heap for various reasons, such as:
    - When a variable is returned from a function, it needs to survive beyond the function's lifetime, so it's allocated on the heap.
    - When a variable is stored in a data structure (like a slice or a map) that lives beyond the function's scope, it may be allocated on the heap.
    - When a variable's address is taken using the `**&**` operator and stored in a pointer, it may need to be allocated on the heap because it can be accessed from other parts of the program.
5. **Performance Implications:** Allocating memory on the heap involves more overhead than allocating it on the stack, as it requires dynamic memory management. Therefore, minimizing heap allocations and keeping data on the stack whenever possible can lead to more efficient code.

In summary, "escaping to the heap" in Go refers to the process of a variable being allocated on the heap instead of the stack due to its usage patterns and lifetime requirements. The Go compiler uses escape analysis to make these determinations and optimize memory allocation. Understanding escape analysis can help Go developers write more efficient and performant code.

  

# Reflection capabilities

  

  

  

# Conclusion

In this document, we have discussed two key features of Go's concurrency model: channels and goroutines. Goroutines are lightweight threads of execution that are managed by Go's runtime, and channels are typed conduits through which you can send and receive values between goroutines. By using these powerful concurrency features, you can write efficient and scalable concurrent programs in Go.

[[HTTP package]]

  

/pave