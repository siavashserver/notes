---
title: Locks
---

## Common Lock Types

### (X) Exclusive Lock

Acquired by *data modification* operations (INSERT, UPDATE, DELETE) on table rows. Only one active *Exclusive Lock* can be set on a row; No *Shared Lock* allowed.

### (S) Shared Lock

Acquired during read operations (SELECT) on table rows. Multiple *Shared Locks* can be acquired on a row; No *Exclusive Lock* allowed.

### (U) Update Lock

Acquired during search for applicable rows (*WHERE*, etc), and turned into *Exclusive Lock* if satisfying the given predicate, otherwise released.

### (I*) Intent Lock

*IX, IS, IU* locks are acquired on table level, so that during table modification, there won't be any need to scan all rows for possible locks.

### Range Lock

Acquired on all possible rows satisfying the given predicate, during *Serializable Transaction Isolation Level*.

## Deadlock

Occurs when two or more transactions block each other's needed resources,
resulting in perpetual waits. Databases detect deadlocks via wait-for graphs and
cycles, terminating one transaction to break the impasse. Minimizing deadlocks
demands consistent locking order, short transactions, and in some cases, using
row versioning or snapshot isolation (with its own trade-offs).

## Locking Methods

### Pessimistic Locking

Locks the specified rows using `SELECT ... FOR UPDATE` from modification by other transactions.

```sql
-- Begin transaction
BEGIN;

-- Lock selected rows
SELECT balance
FROM users
WHERE user_id = 123
FOR UPDATE;

-- Safely perform update
UPDATE users
SET balance = balance + 100
WHERE user_id = 123;

COMMIT;
```

### Optimistic Locking

Involves *row versioning* and manual conflict management.

```sql
-- Assume current_version is the version number retrieved by the initial SELECT statement

DECLARE @current_version INT;
SELECT @current_version = version FROM employees WHERE id = 1;

IF EXISTS (SELECT 1 FROM employees WHERE id = 1 AND version = @current_version)
BEGIN
    UPDATE employees
    SET name = 'Alice Smith', position = 'Senior Engineer', version = version + 1
    WHERE id = 1 AND version = @current_version;

    IF @@ROWCOUNT = 0
    BEGIN
        -- Handle conflict (e.g., retry or inform the user)
        PRINT 'Conflict detected. The row has been modified by another transaction.';
    END
END
ELSE
BEGIN
    -- Handle case where the row does not exist or version does not match
    PRINT 'The row does not exist or the version does not match.';
END
```
