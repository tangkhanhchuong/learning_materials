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
    - Is insert and random access operation always O(1)? -> random access: O(1), insert: armotized O(1) due to resizing when run out of capacity
        -> Follow up: Prove that insert is armotized O(1) (using formular)
        -> Is it armotized O(1) if we resize 3x, ... 10x?
3. Linked List
    - Compare Array List and Linked List. When to use each other? -> notice about memory layout
    - Implement LRU, LFU
    - Prove fast and slow pointers technique
4. String
    - How to compare 2 strings efficiently?
    - What is the difference between string and dynamic array?
    - What is `string pool`?
5. Stack and Queue
    - How to implement Stack using Queue?
    - How to implement Queue using Stack? 
    -> Time and space complexity
    - Implement Queue using Array -> Circular Deque (Ring Buffer)
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
    - Why memorization reduce time complexity?
12. Binary Search
    - Why time complexity of BS is O(logn)? Prove that
13. Heap
14. Trie    

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

## NETWORKING

## DATABASE
1. Compare `SQL` and `NoSQL`?
    - How do they scale up?
    - How is transaction handled?
    - What is `ACID`? Why `NoSQL` is `eventual consistency`?
    - `CAP` Theorem
2. What is `parameterized statement` / `prepared statement`? How does it work manually?
    - What is `SQL injection`? how to avoid it?
    - How many `requests` you have to send to Database in a single `prepared statement` query? // one for compile and one for execute
    - Can you reuse the compiled query multiple times? (does it help to speed up your application?)
3. How `indexing` works internally?
    - Which algorithm and data structure does `index` use? Why?
    - How does `composition index` work?
    - How to know whether your query using `index` or not?
    - How index work in this case: WHERE age = 5 and Where age > 5? The complexity to go to the next record?
    - How `index` work with `char`?
4. Complexity of SQL query? How to measure it?
    - What is the difference between `EXPLAIN` and `EXPLAIN ANALYZE
    - How SQL optimize a query?
    - Complexity of this query `SELECT * FROM abc ORDER BY name LIMIT 10 OFFSET 1000000` // SELECT 10 record from offset 10^6 after sort by name (which is a char)? How to optimize it? 
    - What is the complexity of `COUNT(*)` query?
    - How to write query to avoid full scan table?
    - Complexity of `JOIN`, `INNER JOIN` and `OUTER JOIN`
5. What is `Database Replication`? When do we need it?
    - What is `bin log`? How does `Master DB` sync with `Slave DB`?
    - Can a `Slave DB` be a slave of another `Slave DB`? (in-directly, we do not need to sync  from `Master DB`)
6. What is `Database Sharding`? When do we need it?
    - Which rule do we can apply to `DB Sharding`?
    - How to ensure `primary key` globally unique when sharding?
    - How can we  shard a table to multiple tables (same server) and multiple DB (multiple servers)?
    - How does SQL query work when we use sharding? For example, we query the data in different tables?
7. What is database transaction?
    - How do `rollback` work internally?
    - What is `isolation level`? How many of them? `dirty read`, `unrepeatable read`, `phantom read`?
    - How does transaction work when there are many concurrent requests?
    - How to avoid `race condition` in DB? `read lock`, `write lock`?
    - What is `distributed transaction`? How to make a transaction when a DB need to access multiple DB?

## SECURITY
```
