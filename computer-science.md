# Table of Contents

- [DSA](#dsa)
- [OPERATING SYSTEM](#operating-system)
- [NETWORKING](#networking)
- [DATABASE](#database)
- [SECURITY](#security)

## DSA
1. Array
    - How array is stored in mememory?
    - What is the difference between Row-Major vs Column-Major Memory Access? Which is faster to access?
2. Array List (vector in C++, list in Python, slice in Go...)
    - How does it resize when running out of space memory?
    - Why is the resizing array list ratio 2x? Why does it increase linearly or 3x?
    - Is insert and random access operation always O(1)? -> random access: O(1), insert: artimored O(1) due to resizing when run out of capacity
3. Linked List
    - Compare Array List and Linked List. When to use each other? -> notice about memory layout
4. String
    - How to compare 2 strings efficiently?
5. Stack and Queue
    - How to implement Stack using Queue?
    - How to implement Queue using Stack? 
    -> Time and space complexity
6. Hashmap
    - Can we achieve the original value?
    - What is hash collision?
    - How to resize the hashmap when running out of capacity?
    - Implement hashmap from sratch, with resizing mechanism
7. Sorting
    - Compare Quick sort and Merge sort? Time and space complexity
    - Implement Quick sort and Merge sort
    - Heapsort -> Implementation
    - Which algorithm is used in Python, Go, NodeJS, Java?
8. DFS and BFS
    - Compare DFS and BFS
    - Which is better? Give a usecase one has different performance from other
9. Backtracking 
10. Dynamic Programming
11. Binary Search
12. Heap
13. Trie    

## OPERATING SYSTEM
1. `Process` and `Threads`
   - What are the differences between `thread` and `process`? Why they are said that `Thread is a lightweight process`?
   - How does CPU switches between threads/processes (context switching)?
   - Which data does `process` and `thread` need to live? How data is ensured safety? (in case single-CPU core and multiple-CPU core)
   - What is `multi-process` and `multi-thread`? When to choose each?
       + States in `process` and their transition
       + What will happens if a thread is `waiting` or `sleeping`?
       + How CPU detects that a thread is `sleeping` or ready to run?
       + Scheduled algorithm
   - What is `thread-pool`? How to use it? How to create a `thread-pool` in your programming language?
   - Can 2 different `processes` access or change data of each other `address space`?
        + Can 2 `processes` share the same library? How?
        + How does `debugger` work? How it can attach to a running process and change data of that process? (so cool, right?)
    - How can 2 processes communicate to each other? (focus on OS way)
    - What is `child process` and how to create it?
        + What data `child process` have when it is created?
        + Can it read/write data on it `parent process`?
        + What is `copy on write` (COW) (optional)?
        + What will happen if a `child process` changes variable of `parent process`?
2. `Concurrency` and `Parallel`? (in case single-CPU core and multiple-CPU core)
    - What is `critical` zone?
    - What is `race condition` and how to handle it?
    - Locking mechanism? `mutext`, `semaphore`, `spinlock`, `read lock`, `write lock`
    - What is `deadlock` and how to avoid `deadlock`?
3. Memory management
    - What is `memory layout`?
    - What is `memory hierarchy`?
    - What is `heap space` and `stack space`?
    - What is `virtual memory`? Why do we need it? How does it work?
       + How large `virtual memory` is?
       + What is `paging`?
       + Can 2 processes map to the same `physical address`? Where and in which case?
    - What will happen when we open an application?
    - What will happen when we call a function (with parameter) or return from function?
       + What will happen with `stack`? (Why don't we use `heap` here?)
       + What will happen with `register`?
    - What causes `stack overflow`?
    - What is `dynamic allocating`? How does it work?
       + How does `deallocation` work?
       + What happen when your computer is full of memory?
       + Why we don't need to `deallocate` local variables?
    - What is `Garbage Collector`?
        - When is it triggered?
        - How does it work? (in Python, NodeJS, Go...)
    - What is `pointer`? What is the difference between `pass by value` and `pass by reference`?
    - Where is `global variables` saved?
4. System call
    - What is system call? How to do a `syscal`?
    - What happen with CPU when we do a `syscal`?
    - What is the difference between `user space` and `kernel space`?     
```
