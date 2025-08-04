# Table of Contents

- [A. OS Core Architecture](#a-os-core-architecture)
- [B. Process and Thread Management](#b-process-and-thread-management)
- [C. Memory Architecture and Management](#c-memory-architecture-and-management)
- [D. Concurrency and Synchronization](#d-concurrency-and-synchronization)
- [E. CPU Scheduling](#e-cpu-scheduling)
- [F. Input/Output Systems](#f-inputoutput-systems)

# A. OS Core Architecture
## 1. Kernel
- Core part of Operating System
- Sits between hardware and software
	- Managing system resources
	- Allowing software to interact with hardware safely and efficiently
### What the Kernal does
- `Process Management`
	- Scheduling CPU time
	- Creating, pausing and terminating processes
- `Memory Management`
	- Allocating and freezing RAM
	- Managing virtual memory, paging and swapping
- `Device Management`
	- Talking to hardware through device driver
	- Handle I/O operations
- `File System Management`
	- Reading/Writing files
	- Managing permissions and disk structures
- `System Calls`
	- Provides an interface between user programs and the OS (e.g., open, read, write)
### Types of Kernels
- Monolothic Kernel
- Microkernel
- Hybrid Kernel
--- 
## 2. Processor (CPU)
- The **brain** of a computer — it executes instructions from programs and coordinates operations between hardware and software components.
### Key Role
- `Instruct Execution`: Fetches, decodes and executes instructions from memory. Instructions include arithmetic, logic, memory access and control operations
- `Cordination`: Controls and manages data transfers between memory and storage
---
## 3. System Calls
- `A system call` is a way for a user program to request a service from the `operating system kernel`, like **reading a file**, **creating a process**, or **communicating over the network**.
- User programs can’t directly access hardware or kernel resources for safety reasons, so they must ask the OS through system calls.
### How it works
- `User mode → Kernel mode transition`: The **user process triggers a special CPU instruction** (e.g., int 0x80 on x86, syscall on modern CPUs) to switch into kernel mode.
- `Pass parameters`: Arguments (like **file descriptors**, **buffer addresses**) are passed to the kernel (often via registers or a stack).
- `Kernel handles the request`: Executes the privileged operation (e.g., read disk, send data).
- `Return to user mode`: The result is passed back, and control returns to the user program
### Types if system call
- `read()` → read from a file
- `write()` → write to a file
- `fork()` → create a new process
- `exec()` → run a program
- `open()`, `close()` → manage files
- `socket()` → network communication

### Why System Calls matter?
- Provide **a controlled, secure interface** between user apps and hardware
- **Protect OS** from buggy or malicious code
- **Abstract away hardware details** for portability
---
# B. Process and Thread Management
## 1. Process
- An independent program in execution
- `Memory`: Has its own separate memory space
- `Communication`: More complex and slower (via IPC - Inter-process communication)
- `Isolation`: Strongly isolated, 1 process crashing doesn't affect others
- `Creation Overhead`: Heavyweight, less overhead
- `Context Switching`: Slower due to more data to manage
- `Scalability`: More stable due to isolation
- ---
## 2. Thread
- The smallest unit of execution within a process
- `Memory`: Share memory space with other threads of the same process
- `Communication`: Easier and faster due to shared memory
- `Isolation`: Not isolated, errors in 1 threads can affect the others
- `Creation Overhead`: Lightweight, less overhead
- `Context Switching`: Faster because of less data to save/restore
- `Scalability`: Less stable, shared memory can lead to synchronization issues
---
## 3. Multi-threading Pros and Cons
### 3.1. Pros:
- `Improve Performance`: Ultilizing Parallelism. Threads can run concurrently, especially on multi-core CPUs -> handle CPU-bound and I/O-bound tasks more efficiently
- `Resoure sharing`: Threads share the same memory space (within a process), reducing need for IPC -> Faster and more efficient data exchange compared to multiple processes
- `Efficient Use of Resource`: Threads are lighter than processes, requiring less memory, overhead and faster in context switching
### 3.2. Cons
- `Complexity`:
	- Writing and debugging multi-threaded code is harder due to the need for synchronization
	- Bugs for race condition, deadlock and livelock can be subtle and hard to reproduce
- `Resource Contention`: Too many threads can lead to competition for CPU, memory, or I/O -> reduce performance (thrashing)
- `Platform and Language Limitations`:
	- Some languages don't allow true parallelism in threads.
	- Thread behaviour can be vary between operating systems
---
## 4. Context Switching
`Context switching` is the process where the CPU stops working on one task (thread/process) and starts working on another, saving the current task's state so it can resume later.
### What Happens During a Context Switch?
- `Save current state` of the CPU (registers, program counter, etc.) to memory.
- `Load the saved state` of the new thread/process into the CPU.
- `Resume execution` of the new task from where it left off.
### Benefits of Context Switching
- `Enables multitasking`: Allows many programs or threads to run as if simultaneously.
- `Improves responsiveness`: Ensures high-priority tasks get CPU time.
- `Fairness`: Helps allocate CPU time across all threads or processes.
---
# C. Memory Architecture and Management
## 1. Memory Hierarchy
###  Volatile
- `CPU Register`: Smallest and fastest storage in CPU, very fast and hight cost
- `L1, L2, L3 Cache`: On-chip/off-chip memory used to store frequent data, fast and high cost
- `RAM`: Main system memory for programs and data, moderate in both speed and cost
### Non-volatile
- `SSD/HDD`:	Persistent storage for OS, files, and programs, slower and lower cost
- `Remote Storage`:	Network-based storage (e.g., cloud, DBs), very slow, cost can be varied
---
## 2. Virtual Memory
- `Virtual memory` is a memory management technique that gives each process the illusion of its own large, private memory space.
- The OS + hardware map `virtual addresses` to physical RAM
- Enable memory protection and multitasking
- Allow running programs larger than physical memory
---
## 3. Paging
- Virtual and Physical memory are split into `fixed-size blocks`:
	- `Pages` (virtual memory)
	- `Frames` (physical memory)
- The OS uses a `page table` to map `pages` to `frames`
- Enable efficient and flexible in memory allocation
---
## 4. Swapping
- When RAM is full, the OS moves **inactive pages** to **disk** (`swap space`)
- Free up RAM for active tasks
- Slower than RAM but allows more program to run
---
## 5. Stack and Heap Memory
### 5.1. Stack Memory
- `Allocation`: In contigous blocks. Each thread has its own stack
- `De-allocation`: Automatic by computer instruction
- `Flexibility`: Smaller, Fixed-size, data structure is linear
- `Typical Errors`: Stack overflows
- `Usage`: Store local primitive variable and function call

### 5.2. Heap Memory
- `Allocation`: Share across threads
- `De-allocation`: Manual by programmer or garbage collector
- `Flexibility`: Larger, Resizable, data structure is hierarchical
- `Typical Errors`: Memory leaks
- `Usage`: Store objects, dynamic memory and reference type
---
## 6. Garbage Collector
- Part of the runtime environment (like JVM or Python interpreter)
- **Automatically manages memory** by **de-allocating** objects that are no longer in use
- Prevent **memory leaks** and reduce burden of **manual memory management**

### 6.1. Javascript
- **GC Type**: `Mark-and-Sweep` 
- **How It Works**:
	- **Roots** (like global variables) are scanned.
	- **Reachable objects** are marked.
	- **Unreachable objects** are collected (swept).
	- Uses **generational GC** (short-lived vs long-lived objects).
	- Runs automatically during idle time to avoid lags.

### 6.2. Python
- **GC Type**: `Reference Counting` + `Cycle Detector`
- **How It Works**:
	- Every object has a reference count.
	- When count hits zero, it's immediately collected.
	- But: cycles (like `a ↔ b`) aren’t handled by refcount alone.
	- Python’s GC detects and cleans cyclic references (in container types like lists, dicts, etc).
	- We can trigger GC manually using `gc.collect()`

### 6.3. Golang
- **GC Type**: `Concurrent`, `Mark-and-Sweep`, non-generational (since Go 1.5+)
- **How It Works**:
	- Runs concurrently with application (low pause).
	- **Mark phase**: Find reachable objects.
	- **Sweep phase**: Free memory of unreachable ones.
	- Designed for low latency and high throughput.
	- We can tune GC via GOGC (e.g., GOGC=100 controls how often GC runs).
---
# D. Concurrency and Synchronization
## 1. Race condition
**A race condition** happens when **two or more threads access shared data at the same time**, and the **final result depends on the timing** of their execution.
### How to prevent and resolve
- Use `locks`/`mutexes` to ensure only one thread modifies shared data at a time.
- `Atomic operations`
- `Avoid shared state`
- `Thread-safe data structure` (like `ConcurrenyHashMap`)
---
## 2. Deadlock
- A situation where **two or more processes are unable to proceed** because each is **waiting for the other to release resources**.
### Condition for Deadlock
- `Mutual Exclusion`: A thread or process holds a non-sharable resource — only one thread/process can use the resource at a time.
- `Hold and Wait`: A process is holding at least one resource and waiting to acquire additional resources that are currently held by other processes.
- `No Preemption`: A process cannot be forced to release its resources; it must release them voluntarily
- `Circular Wait`: A set of processes are waiting in circular chain, where each process is waiting for a resource held by the next process in the chain 

### How to prevent and resolve
- `Avoid Circular Wait`: Ensure that all the processes acquire resources in pre-defined order
- `Timeout`: Let processes gives up waiting after a certain timeout and retry
- `Resource Preemption`: Allow processes to be forcibly release their resources to avoid deadlock
---
## 3. Mutexes
- A **lock** that allows **only one thread to access** a critical section at a time.
- If one thread holds the mutex, others must wait.
### Use When
- We want to protect shared data from concurrent access.
---
## 4. Semaphore
- A `counter-based lock`:
	- `Counting semaphrore`: Allows N threads to access a resource
	- `Binary semaphore` (like `mutext`): 
### Use When
- We want to limit concurrent access (e.g., max 3 threads to DB).
- We need signaling between threads (like a flag).
## 5. Monitor
- A **high-level abstraction** that combines:
	- A `mutex` (for mutual exclusion)
	- `Condition variables` (for signaling/waiting)
### Use When
- We want `automatic locking` + `waiting/notification` (e.g., with `wait()` and `notify()`).
---
# E. CPU Scheduling
## 1. Purpose of CPU Scheduling
- `Maximize CPU ultilization`: Keep the CPU busy as much as possible
- `Improve throughput`: Complete more processes in less time
- `Minimize waiting time and response time`: Reduce how long processes wait in the ready queue
- `Ensure fairness`: All processes get a chance, avoiding starvation
- `Prioritize critical tasks`: Let high-priority or real-time tasks run first when needed
---
## 2. Common algorithms
- `FCFS (First-Come, First-Serve)`: Processes are scheduled in the order they arrive.
	- **Pros**: Simple, fair
	- **Cons**: Can cause long wait times (convoy effect)
- `SJF (Shortest Job First)`: Process with the **shortest burst time** is scheduled next.
	- **Pros**: Optimal average waiting time
	- **Cons**: Hard to predict job length; risk of **starvation**.
- `Round Robin (RR)`: Each process gets a **fixed time slice** (quantum), then moves to the back of the queue
	- **Pros**: Good for time-sharing systems.
    - **Cons**: Performance depends on quantum size.
- `Priority Scheduling`: CPU is given to the **highest-priority** process.
	- **Pros**: Useful for real-time systems
	- **Cons**: Low-priority processes may starve -> Can be **preemptive** or **non-preemptive**.
---
# F. Input/Output Systems
## 1. Blocking I/O
- The program waits (gets "blocked") until the I/O operations is complete
- CPU does nothing else in the meantime
### Pros
- Simple, easier to write and understand
### Cons
- Watse CPU if waiting for slow devices (disk or network)
- Not suitable for high-performance or real-time systems
---
## 2. Non-Blocking I/O
- The program doesn't wait. If I/O isn't ready, it moves on and retries later
### Pros
- CPU can do other tasks while waiting
- More scalable
### Cons
- More complex code (need polling or callbacks)
- Can result in **busy waiting** (checking again and again)
