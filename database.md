# Table of Contents

- [A. TRANSACTIOn](#a-transaction)
- [B. LOCKING](#b-locking)
- [C. CONCURRENCY CONTROL](#c-concurrency-control)
- [D. INDEXING](#d-indexing)
- [E. POSTGRESQL SQL EXECUTION & OPTIMIZATION](#e-postgresql-sql-execution--optimization)
- [F. REPLICATION & BACKUP](#f-replication--backup)

# A. Transaction

## 1. SQL Isolation Levels

### 🔍 Why Isolation Levels Matter
- Control how transactions interact when accessing shared data.
- Help balance **consistency**, **performance**, and **concurrency**.
- Defined by the SQL standard and implemented differently in each DBMS.

---

### ⚠️ Common Concurrency Problems

| Problem               | Description                                                                 | Example                                                                 |
|-----------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------------------|
| **Dirty Read**         | Reading uncommitted data from another transaction                          | Tx1 updates row A, Tx2 reads it before Tx1 commits (or rolls back)     |
| **Non-repeatable Read**| Reading the same row twice yields different results                         | Tx1 reads row A, Tx2 updates it, Tx1 reads again and sees new value    |
| **Phantom Read**       | A transaction sees new rows upon re-querying with the same predicate        | Tx1: `SELECT * WHERE age > 30`; Tx2 inserts matching row; Tx1 sees it  |
| **Write Skew**         | Two concurrent transactions make conflicting updates violating constraints  | Scheduling doctors — both Tx1 and Tx2 validate, then update separately |

---

### 🔒 Isolation Levels and Their Guarantees

| Level                | Prevents                         | Allows                        | Notes                                                                 |
|----------------------|----------------------------------|-------------------------------|-----------------------------------------------------------------------|
| **Read Uncommitted** | -                                | Dirty read, Non-repeatable, Phantom | Almost no isolation. Fastest, rarely used.                         |
| **Read Committed**   | Dirty read                       | Non-repeatable, Phantom       | Default in PostgreSQL. Each read sees committed data.                |
| **Repeatable Read**  | Dirty, Non-repeatable            | Phantom, Write skew           | All reads repeatable. Phantom rows still possible.                   |
| **Serializable**     | Dirty, Non-repeatable, Phantom, Write skew | -                         | Strictest. Emulates serial execution using predicate locks (in PG).  |

---

### 📌 PostgreSQL Specifics

- **Default Level**: `Read Committed`
- **Serializable** uses **Serializable Snapshot Isolation (SSI)**:
  - Uses predicate locks to detect conflicting reads/writes.
  - Fails transactions with serialization errors (instead of blocking).
- `SELECT ... FOR UPDATE` promotes row locks manually for consistency.

---

### ✅ Summary

| Isolation Level     | Dirty Read | Non-repeatable Read | Phantom | Write Skew | Notes                          |
|---------------------|------------|----------------------|---------|-------------|--------------------------------|
| Read Uncommitted    | ❌         | ❌                   | ❌      | ❌          | Lowest safety, highest speed   |
| Read Committed      | ✅         | ❌                   | ❌      | ❌          | Safe for most apps             |
| Repeatable Read     | ✅         | ✅                   | ❌      | ❌          | Still allows phantom reads     |
| Serializable        | ✅         | ✅                   | ✅      | ✅          | Highest safety, possible retries |

```sql
-- Example: Forcing serializable in PostgreSQL
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```


# B. Locking
## 1. What is Database Locking?
- A **mechanism** used to **control concurrent accesses** to data in database
- Ensuring **data integrity and consistency** when multiple transactions try to read or modify the same data

## 2. Why do we need Locks?
- Prevent **data corruption** when multiple transactions operate on the same rows or table
- Maintain the **ACID** properties (especially isolation) in transaction
- Handle **race conditions**, such as two transactions updating the same records 

## 3. Locking Modes
- `Optimistic Locking`: Assumes conflicts are rare, checks conflicts before commit
- `Pessimistic Locking`: Locks data during the transaction

## 4. Lock Levels
- `Row-Level Lock` -> One row
- `Page-Level Lock`: Groups of rows -> Physical page
- `Table-Level Lock` -> Entire table
- `Database-Level Lock` -> Entire database

## 5. Lock Types:
### 5.1. Shared Lock
- **Scope:** Entire table
- **Purpose:** Protect table structure during reads
- **Behaviour:**
	- Allows multiple `shared locks` (reader)
	- Blocks any writer that need an exclusive lock on the table
### 5.2. Exclusive Lock
#### a. Table-Level (`ALTER TABLE`, `DROP TABLE`)
- **Scope:** Entire table
- **Behaviour:**
	- Block both readers and writers
#### b. Row-Level (`SELECT ... FOR UPDATE`)
- **Scope:** Specific rows only
- **Purpose:** Prevent other transactions from modifying those rows
- **Behaviour:**
	- Block other writers on the same rows
	- Allow readers (Reader can see older commited version of this row, thanks to MVCC)
	- Other rows in the table are unaffected

### 5.3. Advisory Lock
- Application lock
- Preventing duplicate logic execution
- Don't protect data

### 5.4. Predicate Lock
- A logical lock on a condition (predicate) instead of a specific row or range.
- Play only under `SERIALIZABLE` isolation level (not under `READ COMMITTED` or` REPEATABLE READ`).
- All `SELECT` queries acquire `Predicate Lock` under `SERIALIZABLE`

### 5.5. Schema Lock
- Protect table definitions and metadata from concurrent changes
- Automatically acquired during DDL (ALTER TABLE, DROP TABLE).

## 6. Deadlocks
### 6.1. Conditions
- `Mutual Exclusion`: A thread or process holds a **non-sharable resource** — only one thread/process can use the resource at a time.
- `Hold and Wait`: A process is holding at least one resource and waiting to acquire additional resources that are currently held by other processes.
- `No Preemption`:  A process cannot be forced to release its resources; it must release them voluntarily
- `Circular Wait`: A set of processes are waiting in a circular chain, where each process is waiting for a resource held by the next process in the chain.
### 6.2. Prevent Strategies:
- `Avoid Circular Wait`: Ensure that all processes acquire resources in a predefined order.
- `Timeout`
- `Resource Preemption`: Allow a process to be forcibly stripped of resources to avoid deadlock.
- `Wait-Die or Wound-Wait Schemes`: These schemes use priority and time-stamping to prevent circular waits.


### 6.3. How to detect:
Use `pg_locks` for debugging deadlocks or blocked queries
```sql
SELECT * FROM pg_locks pl JOIN pg_stat_activity sa ON pl.pid = sa.pid;
```

# C. Concurrency Control
## 1. Postgres

### 1.1. MVCC
- Postgres's mechanism for handling concurrent transaction without blocking reads and write unnecessarily -> `Isolation` and `Consistency`
- Instead of locking rows for read, every row (tuple) in PostgreSQL has hidden system columns:
	- **ctid**: uniquely identifies the physical location of a row in the table (**block_number**, **tuple_index**)
		- **block_number**: which 8KB page the row is stored in
		- **tuple_index**: which slot (row number) in that page
	- **xmin**: the transaction ID that inserted the row
	- **xmax**: the transaction ID that deleted or updated the row (if any)
- A row is visible to transaction if:
	- `xmin is committed`
	- AND  `xmin <= T.xid`
	- AND
		- `xmax = 0` - the row hasn't been deleted
		- OR `xmax is not commited` - delete is still in-progress (row is still available)
		- OR `xmax > T.xid` - delete happens after your transaction starts, so row is visible to you.
### How it works
- When a row is updated:
	- PG does not overwrite the old row, instead it create a new row with new `xmin`
	- The old version gets `xmax` set to updating transaction ID
- `DELETE` works similarly (it sets `xmax` but does not physically remove the row)
- Old versions remain in the table until `VACUUM` removes them

### 1.2. VACUUM
- **VACUUM**:
	- Run frequently
	- Remove dead rows
	- Free up space
	- Run concurrently with other processes
	- Doesn't block rows
- **VACUUM FULL**:
	- Manually clean up
	- Rewrite the entire table to new disk
	- Physically shrinks the file on disk.
	- Shrink the file on disk
	- Block all access during the operations
- **VACUUM** is the process that cleans up those dead rows — but only when they’re no longer visible to any transaction.

### 1.3. Snapshot
- A snapshot is a consistent view of the database at a point in time.
- It’s part of PostgreSQL’s `MVCC` (Multi-Version Concurrency Control), used to isolate transactions from each other.
- Each transaction sees a snapshot:
  - Its own transaction ID
  - A list of active transactions at the time it started
  - A list of committed transactions before it started

# D. Indexing

## 1. ✅ B+ Tree Index (Default)

- A **balanced tree** structure (all leaves at the same level)
- **Internal nodes** guide search; **leaf nodes** contain all actual keys
- **Leaf nodes are linked** → fast range scans
- Default indexing method in PostgreSQL

### Why B+ Tree?

- High fan-out (300–400 in PostgreSQL) → shallow depth → few disk I/Os
- Ideal for:
  - Equality queries: `=`
  - Range queries: `<`, `>`, `BETWEEN`
  - Sorting: `ORDER BY`, `MIN`, `MAX`

- A balanced tree data structure (all leaves at the same level)
- Optimized for disk-based storage (like in PostgreSQL indexes).
- All keys exist in leaf nodes; Internal nodes only guide the search.
- Leaf nodes are linked for fast range queries.
- Order = m (max number of children per node).
	- Max children = m
	- Leaf nodes are connected in a linked list for sequential access
### How it works
#### a. Search
- Start at root.
- Compare key with internal node keys.
- Follow the correct child pointer.
- Repeat until reaching leaf.
- Find key (or determine it’s missing).
#### b. Insert
- Search for correct leaf.
- Insert key in sorted order.
- If leaf is full:
	- Split leaf into two nodes.
	- Push middle key up to parent.
- If parent overflows, repeat split upward.
- Keeps tree balanced.
#### c. Delete
- Find key in leaf and remove it.
- If underflow occurs, borrow from sibling or merge nodes.
- Adjust parent keys if needed.

#### d. Why B+ Tree for Databases?
- High fanout (300–400 in PostgreSQL) → very few levels → few disk I/O.
- Range queries are fast because leaves are linked.
- Minimizes random disk reads by clustering keys.

---

## 2. ⚙️ PostgreSQL Index Scan Types

### a. Index Scan

- Uses one index to get TIDs (row pointers)
- Fetches actual rows from the heap
- Good when:
  - Selectivity is high
  - Query uses indexed columns only

### b. Bitmap Index Scan

- Can combine **multiple indexes**
- Steps:
  1. Each index produces a **bitmap of TIDs**
  2. Bitmap operations (e.g., AND, OR)
  3. Final TIDs used to fetch heap rows
- Efficient when:
  - Query has **multiple conditions**
  - Many rows match but not too many to make heap access expensive

### c. Index-Only Scan

- Heap access **skipped** entirely if:
  - All required columns are in the index (covering index)
  - Tuple visibility is confirmed using the **visibility map**
- Very fast; avoids disk I/O

---

## 3. 🛠️ Advanced PostgreSQL Index Features

### Composite Indexes

```sql
CREATE INDEX ON users (age, country);
```

- Follows **left-most prefix** rule:
  - Works for `age` or `age, country`
  - Not for `country` alone

---

### Partial Indexes

```sql
CREATE INDEX idx_active_users ON users(email)
WHERE is_active = true;
```

- Index only on rows matching the condition
- Saves space and write cost
- Ideal when:
  - Condition is common
  - Data is skewed (e.g., most users are inactive)

---

### Clustered Index

```sql
CLUSTER users USING idx_age;
```

- Reorders the **physical layout** of table rows to match index
- Improves performance for range and sorted queries
- Takes an **exclusive lock**
- Acts like a full `VACUUM`: resets statistics, removes dead tuples

---

### Index-Only Scan (Covering Index)

- Requires:
  - All selected columns are in the index
  - Tuple visibility check succeeds via **visibility map**
- Avoids accessing heap → very fast

---

## 4. 🔍 Index Strategy Summary

| Feature            | Use Case                                     |
|--------------------|-----------------------------------------------|
| B+ Tree            | General-purpose indexing                      |
| Bitmap Scan        | Multi-condition filtering                     |
| Index-Only Scan    | Read-heavy queries on covering indexes        |
| Partial Index      | Data skew, conditional access                 |
| Composite Index    | Multi-column filters with prefix matching     |
| Clustered Table    | Range/sorted access with low update frequency |


# E. PostgreSQL SQL Execution & Optimization

## 1. SQL Logical Execution Order

The **logical** order in which SQL clauses are processed (not written):

1. `FROM`
2. `JOIN`
3. `WHERE`
4. `GROUP BY`
5. `HAVING`
6. `SELECT`
7. `ORDER BY`
8. `LIMIT / OFFSET`

---

## 2. PostgreSQL Query Execution Pipeline

Each SQL statement in PostgreSQL goes through 4 main stages:

1. **Parser**  
   Checks syntax and structure.

2. **Rewrite System**  
   Applies rules (e.g. for views, rules).

3. **Optimizer (Planner)**  
   Generates multiple query plans and picks the cheapest one based on cost estimates.

4. **Executor**  
   Executes the selected plan and produces the final result.

---

## 3. Join Algorithms

| Join Type     | Description   | Time Complexity           | Best Use Case                              |
|---------------|---------------|---------------------------|---------------------------------------------|
| Nested Loop   | Brute-force   | O(n × m)                  | Small datasets, indexed nested joins        |
| Hash Join     | Uses hash map | O(n + m)                  | Equi-joins with no useful indexes           |
| Merge Join    | Sorted merge  | O(mlogm + nlogn + m + n)  | Sorted data or index-based large datasets   |

---

## 4. Set Operations: UNION vs UNION ALL

- **UNION**
  - Removes duplicates (uses `DISTINCT`)
  - Slower due to sorting or hashing
  - Higher memory/CPU usage

- **UNION ALL**
  - No deduplication
  - Faster and more efficient
  - Good for large datasets when duplicates are OK

---

## 5. Index Usage Strategies

---

### 5.1. Using Multiple Indexes Together

```sql
CREATE INDEX idx_age ON users(age);
CREATE INDEX idx_country ON users(country);

EXPLAIN SELECT * FROM users WHERE age < 30 AND country = 'US';
```

PostgreSQL can:
- Use a single index (**Index Scan**), or
- Combine multiple indexes using a **Bitmap Scan**

---

### 5.2. Index Scan vs Bitmap Scan

#### 🔹 Index Scan
- Uses one index (e.g., `age`)
- Fetches TIDs → fetches from heap → applies other filters
- May access many rows unnecessarily

#### 🔸 Bitmap Index Scan
- Uses multiple indexes (e.g., `age`, `country`)
- Combines TIDs via **Bitmap AND**
- Fetches only matching rows from heap
→ More I/O efficient, especially with multiple filters
#### Step
- Scans multiple indexes separately and collects matching TIDs (tuple IDs = pointers to rows).
- Builds bitmaps (in memory) to represent which rows matched.
- Performs bitmap operations (AND, OR) to combine results.
- Sorts and deduplicates TIDs.

Accesses the heap only for the final matching TIDs.
---

### 5.3. Index-Only Scan

- Skips accessing heap — reads only from index
- Requires **covering index** (query columns fully in index)
- **Fastest** option when available

---

### 5.4. Partial Index

Index only rows matching a condition:

```	sql
CREATE INDEX idx_active_users ON users(email)
WHERE is_active = true;
```

#### ✅ Benefits
- Smaller index size
- Faster writes and maintenance
- Great for skewed data (e.g. mostly inactive users)
- Use when condition is frequently queried

---

### 5.5. Clustered Table

Physically reorder rows based on an index:

```sql
CLUSTER users USING idx_created_at;
```

- Speeds up range/index queries
- Requires **ACCESS EXCLUSIVE** lock
- Acts like a `VACUUM FULL`

#### 📌 When to Use
- Table has frequent range queries
- OK with downtime for reclustering

---

## 6. Query Plan Analysis

Use `EXPLAIN ANALYZE` to analyze performance:

```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE age < 30 AND country = 'US';
```

Check:
- Chosen join algorithm
- Index usage
- I/O vs CPU time

---

## 📌 Summary: Indexing & Execution Tactics

| Technique         | Use Case                                  |
|------------------|--------------------------------------------|
| Index Scan        | Simple conditions, 1 index                 |
| Bitmap Scan       | Multiple indexes, multi-condition filters  |
| Index-Only Scan   | All columns in index, avoids heap access   |
| Partial Index     | Skewed data, common conditional filters    |
| Clustered Table   | Frequent range or sorted queries           |

---

Let me know if you'd like a diagram version or printable cheat sheet!

# F. Replication & Backup
## 1. WAL
- WAL is a durability mechanism used to ensure that changes to the database are not lost, even in the event of a crash.
- "Write-ahead" means: log first, apply later.
- WAL contains `binary change records`, not SQL statements.
	- These are low-level instructions like:
		- "Insert this row at offset X on page Y"
		- "Mark this tuple as dead"
		- "Split this B-Tree page"
	- Compact and fast to write and replay.
- WAL is append-only
- PostgreSQL only returns COMMIT to the client after WAL is safely written to disk

## 2. Replication Types
### 2.1. How Streaming Replication Works:
- Primary writes WAL to disk
- WAL is sent over TCP to replicas
- Replicas replay WAL to mirror changes
- PostgreSQL doesn’t allow writes on replicas (unless you promote it).
### 2.2. Logical Replication (Postgres 10+)
- Uses pglogical or built-in CREATE PUBLICATION/SUBSCRIPTION
- Supports:
	- Selective tables
	- Column filtering
	- Version upgrades
- Good for migrations, replica sharding
### 2.3. Synchronous
- Waits for replica to confirm before commit
- Data safety (no data loss)
### 2.4. Asynchronous
- Primary doesn't wait for replica
- Faster commit, but may lose data on crash

## 3. Two Main PostgreSQL Backup Types:
### 3.1. Logical Backup
- Uses SQL dump
- Tools: pg_dump, pg_restore
- ✅ Easy to restore, selective
- ❌ Slower, not great for large DBs

### 3.2. Physical Backup (Base Backup)
- Copies entire data directory + WAL
- Tool: pg_basebackup
- ✅ Fast recovery, supports PITR
- ❌ Needs more storage + WAL archiving


