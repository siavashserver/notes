---
title: Glossary
---

### Database (DB)
- Definition: A structured collection of data stored for retrieval and manipulation.  
- Why it matters: Start answers by clarifying whether you mean an operational DB (OLTP) or analytical DB (OLAP) and the expected workload.

---

### Database Management System (DBMS)
- Definition: Software that stores, manages, and retrieves data for applications; it provides an abstraction over files and storage and implements query, transaction, and security features.  
- Why it matters: Use DBMS to discuss general responsibilities (storage engine, query processing, transactions, backup/restore) and to contrast with specific systems (MySQL, Postgres, MongoDB).

---

### Relational DBMS (RDBMS)
- Definition: A DBMS that organizes data into tables with rows and columns and uses SQL for querying and data integrity (foreign keys, constraints).  
- Why it matters: For relational design questions, reference normalization, joins, ACID transactions, and SQL dialect differences.

---

### SQL and query languages
- Definition: SQL is the dominant declarative language for relational databases; DDL, DML, DCL, and TCL are its common subsets (schema, data manipulation, access control, transaction control).  
- Why it matters: Interviewers expect fluency in writing SELECTs, joins, aggregates, window functions, and explaining when to use DDL vs DML.

---

### NoSQL
- Definition: Broad class of non-relational databases (key‑value, document, column-family, graph) designed for flexible schema, horizontal scale, or specific access patterns.  
- Why it matters: Use NoSQL when explaining denormalized models, eventual consistency, and trade-offs vs RDBMS (schema flexibility, scale, query limitations).

---

### ACID vs BASE
- ACID: Atomicity, Consistency, Isolation, Durability — guarantees in many RDBMS transaction models.  
- BASE: Basically Available, Soft state, Eventual consistency — common in distributed, highly available NoSQL systems.  
- Why it matters: Explain trade-offs: strict correctness (ACID) vs availability and partition tolerance in distributed systems (BASE).

---

### CAP theorem
- Definition: In a distributed system, you can choose two of Consistency, Availability, and Partition tolerance; cannot guarantee all three simultaneously during partitions.  
- Why it matters: Use CAP to justify architecture choices (why a system chooses CP vs AP) and to discuss effects on client-visible behavior.

---

### Transactions and isolation levels
- Definition: Transactions group operations into atomic units; isolation levels (Read Uncommitted, Read Committed, Repeatable Read, Serializable) control visibility and anomalies.  
- Why it matters: Explain use-cases for lower isolation (better throughput) and higher isolation (correctness) and name common anomalies (dirty read, non-repeatable read, phantom).

---

### Indexes (B-tree, Hash)
- Definition: Data structures that speed lookups (B-tree variants for range scans, hash for equality).  
- Why it matters: Talk about choose-an-index strategy, index selectivity, covering indexes, and index maintenance costs (writes, storage).

---

### Query planning and EXPLAIN
- Definition: The DB’s optimizer produces an execution plan; EXPLAIN shows the plan and estimated costs.  
- Why it matters: Show how you diagnose slow queries (missing index, bad join order, full table scan) and how to read an EXPLAIN output.

---

### Normalization and denormalization
- Definition: Normalization reduces redundancy through table decomposition; denormalization intentionally duplicates data for read performance.  
- Why it matters: Justify schema design choices by workload: OLTP favors normalization for correctness; high‑read/low‑write workloads may favor denormalization for performance.

---

### Partitioning (sharding) and replication
- Partitioning/sharding: Splitting data across nodes by key to scale writes and storage.  
- Replication: Copying data across nodes for availability and read scaling (synchronous vs asynchronous).  
- Why it matters: Discuss shard keys, rebalancing costs, consistency implications, and replication lag impacts.

---

### OLTP vs OLAP, Data warehouse, ETL
- OLTP: Transactional systems optimized for many small operations.  
- OLAP/Data warehouse: Systems optimized for complex analytical queries and aggregations; data is often loaded via ETL/ELT pipelines.  
- Why it matters: Use this to explain different storage choices (row vs columnar), indexing, and aggregation strategies.

---

### Concurrency control, locks, MVCC, deadlocks
- Concurrency control: Mechanisms to coordinate concurrent access (pessimistic locking vs optimistic / MVCC).  
- MVCC: Multi-Version Concurrency Control lets readers see consistent snapshots without blocking writers.  
- Deadlocks: Cycles of resource waits; resolved by detecting and aborting transactions.  
- Why it matters: Describe how your chosen DB handles concurrency and how to troubleshoot lock contention.

---

### Materialized views, stored procedures, triggers
- Materialized view: Precomputed result set stored for faster reads (requires refresh).  
- Stored procedures/triggers: Server-side code executed inside the DB for encapsulating logic or reacting to changes.  
- Why it matters: Discuss trade-offs: performance gains vs added complexity and maintainability.

---

### Performance metrics and troubleshooting keywords
- Latency, throughput, IOPS, cache hit ratio, query time, explain plan, slow query log, read/write amplification.  
- Why it matters: In interviews, walk through your diagnostic workflow (measure, reproduce, isolate root cause, fix), and reference concrete metrics you’d monitor.

---

### Short interview-ready examples to use
- Schema design: explain primary keys, foreign keys, normalization choices, and index plan.  
- Slow query: mention EXPLAIN → missing index → add covering index or rewrite query.  
- Scale-out trade-off: explain shard-key selection, rebalancing cost, and replication lag implications.
