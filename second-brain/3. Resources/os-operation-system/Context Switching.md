---
tags:
  - os-operation-system
---
## Process vs Thread
reference: [https://www.youtube.com/watch?v=4rLW7zg21gI](https://www.youtube.com/watch?v=4rLW7zg21gI)

Program: **A program is an executable file, It contains the code, or a set of processor  instructions, that is stored as a file on disk. When the code in program is load to memory and be execute by processor → it becomes a process.**

An active process also includes the **resources** the program needs to run. These resources are managed by the operating system. Some examples are **processor registers**, **program counters**, **stack pointers**, **memory pages** assigned to the process for its heap and stack, etc. Each process has its **own memory address space**. One process **cannot corrupt** the memory space of another process.

So what is a thread? A thread is the unit of execution within a process. A process has at least one thread. It is called the main thread. Earlier we mentioned **registers, program counters, and stack pointers** as being part of a process. **It is more accurate to say that those things belong to a thread.**

**Threads within a process share a memory address space**, it is possible to communicate between threads using that shared memory space

The [**Process Control block(PCB)**](https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/) is also known as a Task Control Block. it represents a process in the Operating System. A process control block (PCB) is a data structure used by a computer to store all information about a process. It is also called the descriptive process. When a process is created (started or installed), the operating system creates a process manager.

**CPU and I/O Bound Processes**: If the process is intensive in terms of CPU operations, then it is called CPU bound process. Similarly, If the process is intensive in terms of I/O operations then it is called I/O bound process.
## How many process can create

[https://www.geeksforgeeks.org/states-of-a-process-in-operating-systems/](https://www.geeksforgeeks.org/states-of-a-process-in-operating-systems/)

![[OS 2025-06-15 20.21.35.excalidraw]]

As for threads, modern CPUs support simultaneous multithreading (SMT), which allows multiple threads to run on a single core. The exact number of threads that can run simultaneously on one core depends on the specific CPU architecture and whether SMT (e.g., Intel's Hyper-Threading or AMD's SMT) is supported. In general, **the number of processes and threads that can effectively run on a CPU core is determined by the CPU's architecture**, the operating system's scheduling algorithms, and the workload being executed.

## Context switching

**Difference between Thread Context Switch and Process Context Switch :**

| No. | Thread Context Switch                                                                                                                                                                                                                | Process Context Switch                                                                                                                                                                                                                                 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 0.  | There are fewer states to track, and more importantly, since threads share the same memory address space, there is no need to switch out virtual memory pages, which is one of the most expensive operations during a context switch | The operating system stores the states of the current running process so the process can be restored and resume execution at a later point. It then restores the previously saved states of a different process and resumes execution for that process |
| 1.  | TCS occurs when the CPU saves the current state of the thread and switches to another thread of the same process.                                                                                                                    | PCS occurs when the operating system’s scheduler saves the current state of the running Program (including the state of PCB) and switches to another program.                                                                                          |
| 2.  | TCS helps the CPU to handle multiple threads simultaneously.                                                                                                                                                                         | PCS involves loading of the states of the new program for it’s execution.                                                                                                                                                                              |
| 3.  | TCS does not involves switching of memory address spaces.All the memory addresses that the processor accounts remain saved.                                                                                                          | PCS involves switching of memory address spaces.All the memory addresses that the processor accounts gets flushed.                                                                                                                                     |
| 4.  | Processor’s cache and Translational Lookaside Buffer preserves their state.                                                                                                                                                          | Processor’s cache and TLB gets flushed.                                                                                                                                                                                                                |
| 5.  | Though TCS involves switching of registers and stack pointers, it does not afford the cost of changing the address space.Hence it is more efficient.                                                                                 | PCS involves the heavy cost of changing the address space.Hence it is less efficient.                                                                                                                                                                  |
| 6.  | TCS is a bit faster and cheaper.                                                                                                                                                                                                     | PCS is relatively slower and costlier.                                                                                                                                                                                                                 |

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