---
title: Transactions
---

## START TRANSACTION / COMMIT / ROLLBACK

```sql
-- 1. Begin transaction
START TRANSACTION; -- or BEGIN;

-- 2. Make changes
UPDATE table_name
SET column_name = 0;

-- 3a. Save changes
COMMIT;

-- 3b. Failure
ROLLBACK;
```

## ACID

- Atomicity: All changes within a transaction are fully committed or rolled back. 
- Consistency: All data written to the database adheres to defined rules, constraints, and cascades.
- Isolation: Concurrent execution of transactions leaves the database in the same state as if the transactions were executed sequentially.
- Durability: Guarantees that a committed transaction will remain committed even in the case of system failure. This is achieved through transaction logs that allow the database to recover its state after a failure.

## Concurrency Phenomena

- Dirty Read: A transaction reads uncommitted (dirty) data from other uncommitted transactions.
- Non-repeatable Reads: Subsequent attempts to read the same data from within the same transaction return different results.
- Phantom Reads: Subsequent reads within the same transaction return new rows.

### Write Skew

Arises under snapshot isolation, allowing concurrent transactions to observe
disjoint states. Write skew can occur when two transactions read overlapping
data, then each writes a complementary piece, resulting in a final state that
violates business invariants (e.g., in a medical shift schedule both doctors
simultaneously schedule time off, leaving a shift uncovered). Serializability
isolation can prevent this; snapshot isolation cannot.

### Second Lost Update

It occurs when two transactions are trying to update the same record at the same
time, and the update from the first transaction is lost because the second
transaction overwrites it. Strategies to prevent lost updates include optimistic
and pessimistic locking, increased isolation levels, and atomic write
operations.

## Concurrency Models

- Pessimistic: Assumes that multiple users accessing the same data would all eventually like to modify the data and override each other's changes. (Locks the data on first access)
- Optimistic: Assumes that the chance of simultaneous updates is low. (Detects and rolls back transactions modifiying the same data using row versioning)

## Transaction Isolation Levels

- Note: Only affects the *Shared Lock* behavior.

| Isolation Level  | Dirty Read | Non-repeatable Reads | Phantom Reads | Performance | Lock                                 | Concurrency Model |
|------------------|------------|----------------------|---------------|-------------|--------------------------------------|-------------------|
| READ UNCOMMITTED | Y          | Y                    | Y             | Low         | None                                 | Pessimistic       |
| READ COMMITTED   | -          | Y                    | Y             | Moderate    | Shared Lock during every read        | Pessimistic       |
| REPEATABLE READ  | -          | -                    | Y             | High        | Shared Lock during whole transaction | Pessimistic       |
| SERIALIZABLE     | -          | -                    | -             | Highest     | Range Lock during whole transaction  | Pessimistic       |
| SNAPSHOT         | -          | -                    | -             | *Depends*   | -                                    | Optimistic        |

- SNAPSHOT performance depends on how often write conflicts occur.

### Setting Transaction Isolation Level

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

START TRANSACTION;
--- Make changes
COMMIT;
```
