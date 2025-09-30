---
title: Transaction Scope
---

# TransactionScope in C#: Comprehensive Guide, Scenarios, Patterns, and Interview Preparation

---

## Introduction

In modern enterprise and data-driven application development, **managing
transactional consistency** is a cornerstone for reliability and data integrity.
The .NET framework, through its System.Transactions namespace, provides robust
constructs to coordinate groups of operations into **atomic, consistent,
isolated, and durable (ACID) transactions**. Among these, the `TransactionScope`
class stands out for its **ease of use, versatility, and support for both local
and distributed transactions**. This guide offers a **comprehensive examination
of TransactionScope in C#**, delving into its behaviors, its application in
various scenarios—including nested, distributed, and ambient transactions—and
best practices with technologies such as Entity Framework and ADO.NET. The
report is designed to elucidate nuanced mechanisms, present detailed code
samples, highlight pitfalls, and equip candidates with a powerful set of
**interview questions and answers**.

---

## 1. Fundamentals of TransactionScope in C#

### 1.1. What Is TransactionScope?

The `TransactionScope` class represents a **block of code that participates in a
transaction**. When a scope is created, an ambient transaction is initiated (or
joined, if one already exists), and all resource managers (such as databases)
opened within this block are **automatically enlisted**. The transaction is only
committed if the `Complete()` method is called before leaving the scope and no
exceptions have occurred. If not, the transaction is automatically rolled back.

The snippet below summarizes basic usage:

```csharp
using (TransactionScope scope = new TransactionScope())
{
    // Perform transactional work here
    // e.g., database inserts, updates, file system operations, etc.

    scope.Complete(); // Commit
}
// Implicit rollback if scope.Complete() not called or exception thrown
```

This block is elegant and encapsulates transaction logic such that
commit/rollback concerns are largely transparent to the developer, **fostering
cleaner, more maintainable code**.

### 1.2. Core Principles

- **Ambient Transaction:** An implicit transaction created by TransactionScope,
  detected by all resource managers accessed inside the scope.
- **Enlistment:** Any enlisted resource manager (like SqlConnection) recognizes
  the ambient transaction when opened within the scope.
- **Automatic Coordination:** All operations within the transaction scope are
  committed or rolled back together, regardless of the number of participating
  connections or operations.

**Key properties:**

- **Atomicity:** Guarantees that all operations within the scope commit as one.
- **Consistency:** Ensures that a transaction brings the system from one valid
  state to another.
- **Isolation:** Transactions do not interfere.
- **Durability:** Once committed, changes persist even in case of failure.

These mechanisms provide a powerful abstraction for writing robust transactional
code, suitable for complex business scenarios and multi-tier application
architectures.

---

## 2. TransactionScope Default Behavior and Properties

### 2.1. Default Constructor and Behaviors

The simplest way to instantiate a `TransactionScope` is via its parameterless
constructor:

```csharp
using (var scope = new TransactionScope())
{
    // transactional work
    scope.Complete();
}
```

The default behavior triggers these key settings:

- **TransactionOption:** `Required`—joins existing ambient transaction, or
  starts a new transaction if none.
- **IsolationLevel:** `Serializable` in .NET Framework, `ReadCommitted` in .NET
  Core 2.x+.
- **Timeout:** Default is 1 minute.

This behavior is suitable for most use-cases but may need customization for
performance or integration scenarios, especially where **lock contention or
deadlocks can occur due to a high isolation level**.

### 2.2. Constructor Overloads and Settings

For granular control, developers may provide a `TransactionScopeOption` and a
`TransactionOptions` object, for example:

```csharp
var options = new TransactionOptions
{
    IsolationLevel = IsolationLevel.ReadCommitted,
    Timeout = TimeSpan.FromSeconds(30)
};
using (var scope = new TransactionScope(TransactionScopeOption.Required, options))
{
    // Transactional work
    scope.Complete();
}
```

A table below summarizes significant constructor parameters:

| Parameter                         | Description                                                | Typical Value(s)                      |
| --------------------------------- | ---------------------------------------------------------- | ------------------------------------- |
| `TransactionScopeOption`          | How the scope interacts with existing ambient transactions | `Required`, `RequiresNew`, `Suppress` |
| `TransactionOptions`              | Includes `IsolationLevel`, `Timeout`                       | See below                             |
| `AsyncFlowOption` (>= .NET 4.5.1) | Enables ambient transaction flow across `async/await`      | `Enabled`/`Disabled`                  |

The choice of these parameters affects **transaction composition, performance,
error handling, and distributed transaction behavior**.

---

## 3. TransactionOptions: IsolationLevel and Timeout

### 3.1. IsolationLevel Explained

The `IsolationLevel` determines how data changed by one operation becomes
visible to other concurrent transactions, affecting dirty reads, non-repeatable
reads, and phantom reads. Supported levels include:

- `ReadUncommitted` (may allow dirty reads)
- `ReadCommitted` (default in most RDBMS, prevents dirty reads)
- `RepeatableRead`
- `Serializable` (most restrictive and deadlock-prone, but safest)
- `Snapshot` (if supported by database engine)

Choosing the right level is **crucial for balancing consistency with concurrency
and throughput**. For instance, high-volume applications often favor
`ReadCommitted`, whereas financial operations may demand `Serializable`.

### 3.2. Timeout

The `Timeout` property specifies the max allowed duration for the transaction
before it's aborted. The default is usually 1 minute. Setting appropriate
timeout values is imperative to prevent **long-running, blocking, or hung
transactions** which can hurt performance and scalability:

```csharp
new TransactionOptions { Timeout = TimeSpan.FromSeconds(10) }
```

Effectively tuning these properties **prevents resource starvation and can
minimize deadlocks and escalations in busy systems**.

---

## 4. Nested Transactions with TransactionScope

### 4.1. Conceptual Model

Nested TransactionScopes manage **nested transaction semantics**, which depend
on the `TransactionScopeOption`:

- **Required (default):** Nested scope joins the existing ambient transaction.
  All work is **committed or rolled back together**.
- **RequiresNew:** Nested scope always creates a **new, independent
  transaction**, temporarily suspending the outer scope’s transaction.
- **Suppress:** Temporarily suspends the ambient transaction inside the scope.

The following code demonstrates nested usage:

```csharp
using (var outerScope = new TransactionScope())
{
    // Work 1
    using (var innerScope = new TransactionScope(TransactionScopeOption.RequiresNew))
    {
        // Work 2 (in a separate transaction)
        innerScope.Complete();
    }
    outerScope.Complete();
}
```

This configuration provides **flexibility in controlling the atomicity and life
cycle of logically related yet separably recoverable units of work**.

### 4.2. Behavior Analysis

If an inner scope is marked complete but the outer is not, all work is rolled
back, unless `RequiresNew` is used. With `RequiresNew`, the inner scope is
independent, so a failure in the outer scope doesn't affect committed inner
scope work.

**Design Implications:**

- Proper selection of scope options **avoids unwanted cascading rollbacks or
  premature commits**.
- **Nested Required:** One big atomic group.
- **Nested RequiresNew:** Partial commits allowed, suitable for operations like
  logging or audit trails which must persist even if the main operation fails.

Misunderstanding these semantics is a common pitfall, leading to subtle,
hard-to-debug data inconsistencies.

---

## 5. Distributed Transactions, Escalation, and MSDTC

### 5.1. Distributed Transaction Overview

A distributed transaction spans **multiple resource managers**, such as two
separate database servers. `TransactionScope` handles such scenarios by
**automatically escalating** a local transaction to a distributed one when
required.

**Escalation Triggers**:

- Opening more than one connection to different databases within the same scope.
- Using database connections that do not support promotable single-phase
  enlistment.
- Mixing different providers/resource managers.

When escalation is triggered, **Microsoft Distributed Transaction Coordinator
(MSDTC)** participates, coordinating the commit across systems.

#### Escalation Example

```csharp
using (var scope = new TransactionScope())
{
    using (var conn1 = new SqlConnection(connStr1))
    {
        conn1.Open();
        // Do work
    }

    using (var conn2 = new SqlConnection(connStr2))
    {
        conn2.Open(); // Escalates to MSDTC
        // Do work
    }

    scope.Complete();
}
```

### 5.2. Diagnosing Transaction Escalation

It is crucial to know **whether a transaction will escalate** because
distributed transactions incur overhead and require MSDTC to be
running/configured correctly on participating machines. If MSDTC is unavailable
or misconfigured, transactions may silently fail or throw exceptions.

- **Avoiding unnecessary escalation** is a best practice. If possible, reuse a
  single connection or limit the scope to a single resource.
- **Check connection string, provider, and usage patterns carefully.** Even
  using two `SqlConnections` concurrently to the same database but through
  different connection strings can trigger escalation.

### 5.3. Optimizations and Considerations

- Recent .NET versions and SQL Server versions (2008+) provide **promotable
  single-phase enlistment** to reduce escalation.
- .NET Core and .NET 5+ have platform restrictions; distributed transactions are
  only supported on Windows.

Proper resource, platform, and configuration alignment is **essential for
production stability**.

---

## 6. Exception Handling and Rollback in TransactionScope

### 6.1. Success and Failure Paths

A `TransactionScope` is **only committed when `scope.Complete()` is called and
the scope exits cleanly**. If an exception is thrown, or `Complete()` is never
called, the transaction is **automatically rolled back**.

```csharp
try
{
    using (TransactionScope scope = new TransactionScope())
    {
        // Work...
        scope.Complete();
    }
}
catch (Exception ex)
{
    // Transaction automatically rolled back
    // Handle exception
}
```

**This removes much boilerplate and makes code clearer, but beware:**

- **Do not catch and swallow exceptions inside the scope block without calling
  Complete.** This results in a silent rollback, which is easy to overlook.
- **Resource disposal order** may impact transaction consistency in rare edge
  cases.

### 6.2. Logging and Auditing

Best practices dictate that **exception context and rollback events be logged**
for operational visibility. This allows troubleshooting in scenarios of
intermittent or systemic failures.

---

## 7. TransactionScope with ADO.NET

### 7.1. Integration

When a `SqlConnection` is opened inside a TransactionScope, it is
**automatically enlisted** in the ambient transaction. All subsequent ADO.NET
operations—`ExecuteNonQuery`, `ExecuteScalar`, etc.—are part of the transaction.

```csharp
using (var scope = new TransactionScope())
{
    using (var connection = new SqlConnection(connectionString))
    {
        connection.Open();
        // Command execution
    }
    scope.Complete();
}
// On error or absence of .Complete(), rollback is automatic.
```

**Note:** If a connection is opened _before_ entering the TransactionScope, it
will **not participate** in the transaction.

### 7.2. Mixed Transaction Management

It is advisable to **avoid combining explicit ADO.NET SqlTransaction management
and TransactionScope**, as this may lead to unpredictability in complex cases.
Choose one strategy per transaction boundary whenever possible.

---

## 8. TransactionScope with Entity Framework 6

### 8.1. Entity Framework 6 Native Support

Entity Framework 6 supports three primary approaches for integrating
transactions:

1. **Implicit:** Rely on SaveChanges() to implicitly wrap changes in a
   transaction.
2. **Explicit DbContextTransaction:** Use
   `DbContext.Database.BeginTransaction()`.
3. **Ambient with TransactionScope:** Share a single ambient transaction across
   multiple context instances and external resources.

Example:

```csharp
using (var scope = new TransactionScope())
{
    using (var context1 = new MyDbContext())
    {
        // Work on context1
        context1.SaveChanges();
    }
    using (var context2 = new MyDbContext())
    {
        // Work on context2 (can be same or different database)
        context2.SaveChanges();
    }
    scope.Complete();
}
```

This design is **especially powerful for coordinated operations across databases
or integrating commands outside EF**, such as ADO.NET or direct SQL.

### 8.2. Advantages and Pitfalls

- **Pros:** Simple syntax, automatic escalation, consistent error handling.
- **Cons:** Distributed transactions can be expensive and require MSDTC. Async
  programming can present issues unless flow is explicitly enabled
  (`TransactionScopeAsyncFlowOption.Enabled` in .NET 4.5.1+).

---

## 9. TransactionScope with Entity Framework Core

### 9.1. EF Core Transaction Approaches

Entity Framework Core (EF Core) supports transactions similarly but has some
caveats:

- `DbContext.Database.BeginTransaction()`: Best for single-DbContext,
  single-database operations.
- **Ambient Transactions (`TransactionScope`):** Used when coordination across
  multiple contexts or non-EF code is required.

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    using (var context = new MyDbContext())
    {
        // EF Core work
        await context.SaveChangesAsync();
    }
    scope.Complete();
}
```

### 9.2. Platform and Async Constraints

- As of EF Core 3.x+, **transactions are not always ambient-friendly—consult the
  provider documentation**. E.g., SQLite provider, in-memory provider, etc., may
  not support distributed transactions.
- Cross-context or cross-DbConnection scenarios typically trigger distributed
  transaction escalation if more than one connection is involved.
- **Async/Await:** Use `TransactionScopeAsyncFlowOption.Enabled` to safely flow
  the ambient transaction to async code.

### 9.3. Special Considerations

- Distributed transactions in .NET Core and later are **Windows-only**
  environments.
- **Provider support** (e.g., Npgsql for PostgreSQL) is limited and must be
  confirmed explicitly。
- Careful handling is required to avoid premature connection opening or
  “leakage” outside the TransactionScope block.

---

## 10. Async/Await and TransactionScope

### 10.1. Enabling Async Flow

Prior to .NET 4.5.1, ambient transactions did **not flow across asynchronous
operations**—meaning, work after `await` might not participate in the original
TransactionScope's transaction. This could lead to **subtle and
difficult-to-diagnose bugs in modern async-heavy codebases**.

.NET 4.5.1 introduced `TransactionScopeAsyncFlowOption` to explicitly allow
this:

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    await SomeAsyncDbOperation();
    scope.Complete();
}
```

Forgetting this parameter **results in InvalidOperationException** at runtime.
Whether async transaction is supported also depends on the database provider and
data access technology.

### 10.2. Relevant Caveats

- **Do not access UI elements or thread-affine resources inside TransactionScope
  when using async.**
- **Always test distributed transaction scenarios across real-world workloads**,
  including performance, to avoid surprises.

---

## 11. Logging and Monitoring TransactionScope Transactions

Logging and monitoring transactional operations is essential for **diagnosing
issues, compliance auditing, and performance tracing**. Recommended strategies
include:

- **Log every transaction start and completion/rollback event**, including
  exceptions and elapsed time.
- Use **DiagnosticSource** and **EventSource** in .NET to automatically capture
  transaction activity for operational dashboards.
- On distributed transactions, monitor both local and MSDTC-related events, as
  issues may arise at infrastructure level rather than in application code.

A lack of proper logging often leads to **opaque, hard-to-reproduce data
corruption or availability issues**.

---

## 12. Sample Code

### 12.1. Simple TransactionScope Usage

```csharp
using (var scope = new TransactionScope())
{
    using (var conn = new SqlConnection(connectionString))
    {
        conn.Open();
        SqlCommand cmd = new SqlCommand("INSERT INTO Products (Name) VALUES ('Pen')", conn);
        cmd.ExecuteNonQuery();
    }
    scope.Complete();
}
```

This straightforward pattern ensures that the database operation is wrapped in a
**robust transaction**. Should an exception occur prior to `scope.Complete()`,
changes are rolled back automatically.

### 12.2. Nested TransactionScope Example

```csharp
using (var outerScope = new TransactionScope())
{
    // Primary transactional work
    using (var innerScope = new TransactionScope(TransactionScopeOption.RequiresNew))
    {
        // Audit log or notification task, committed regardless of outer work's success
        innerScope.Complete();
    }
    outerScope.Complete();
}
```

This pattern is ideal for segregating critical operations that should not be
reversed alongside a broader transaction.

### 12.3. Distributed TransactionScope Example

```csharp
using (var scope = new TransactionScope())
{
    using (var conn1 = new SqlConnection(connStr1))
    {
        conn1.Open();
        // Modify DB1
    }

    using (var conn2 = new SqlConnection(connStr2))
    {
        conn2.Open();
        // Modify DB2
    }
    scope.Complete();
}
// MSDTC coordinates atomicity across both databases
```

This code illustrates how joining connections to different databases
**automatically promotes the transaction to a distributed one**, triggering
MSDTC’s involvement for two-phase commit.

### 12.4. Entity Framework with TransactionScope

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    using (var db1 = new MyDbContext1())
    using (var db2 = new MyDbContext2())
    {
        db1.TableA.Add(new A {Value = "Test"});
        db2.TableB.Add(new B {Value = "Test"});
        await db1.SaveChangesAsync();
        await db2.SaveChangesAsync();
    }
    scope.Complete();
}
```

This block demonstrates **coordinating transactional work across multiple
DbContexts or databases** using EF (classic or Core) and async
operations—following best practices with async flow enabled.

### 12.5. Exception Handling in TransactionScope

```csharp
try
{
    using (var scope = new TransactionScope())
    {
        // Business logic
        throw new Exception("Simulated error");
        scope.Complete();
    }
}
catch (Exception ex)
{
    // Transaction automatically rolled back—log details here
}
```

Here, an exception thrown within the TransactionScope **cancels the
transaction**, rolling back any partial work and preserving data consistency.

---

## 13. Best Practices and Common Pitfalls

### 13.1. Best Practices

- **Always call `scope.Complete()` only when all work has succeeded**.
- **Dispose scopes as early as possible,** avoiding unnecessarily long-running
  transactions.
- **Use `TransactionScopeAsyncFlowOption.Enabled` for async code**; otherwise,
  transaction context may be lost.
- **Minimize transaction scope boundaries** to prevent deadlocks and blocking.
- **Log transaction failures and their reasons**, especially in distributed or
  critical operations.
- **Prefer explicit transaction management for batch/ETL operations** for more
  granular control when appropriate.
- **Avoid cross-database transactions if possible** for scalability—distributed
  transactions should be rare, as they have overhead.

### 13.2. Common Pitfalls

- **Forgetting to call `scope.Complete()`**—causing silent rollbacks.
- **Opening connections outside TransactionScope**, meaning they won't
  participate in ambient transactions.
- **Unintended distributed transaction escalation** from opening multiple
  connections or inadvertent resource use.
- **Async operations without enabling async flow**—leading to lost transaction
  context.
- **Insufficient logging,** making it hard to diagnose transactional failures.
- Using TransactionScope on unsupported platforms (e.g., distributed
  transactions on Linux/Unix without proper support).

Avoiding these mistakes is crucial for **application reliability and
maintainability**.

---

## 14. Interview Questions and Answers

Below is a curated set of interview questions and answers derived from practical
scenarios and current technical expectations when working with
`TransactionScope` in C#. Each answer provides both conceptual clarity and
real-world insight.

### 14.1. What is TransactionScope and how does it simplify managing transactions in C#?

**Answer:**  
`TransactionScope` is a class provided by the System.Transactions namespace in
.NET which encapsulates transactional operations within a using block. It
enables _implicit_ creation and management of ambient transactions so that all
operations performed within the scope—such as database queries or updates—are
treated atomically. It simplifies resource enlistment (e.g., connections,
commands) because whenever a compatible resource (like a SqlConnection) is
opened within the scope, it automatically participates in the transaction,
removing the need for manual enlistment or explicit transaction objects.

### 14.2. Explain the difference between Required, RequiresNew, and Suppress in TransactionScopeOption.

**Answer:**

- **Required:** Joins an ambient transaction if one exists; otherwise, starts a
  new one (default).
- **RequiresNew:** Always starts a new transaction, suspending any existing
  ambient one. This is independent from outer transactions.
- **Suppress:** Suppresses any ambient transaction—no transaction is active in
  the block, regardless of the caller context.

Knowledge and correct selection of these options ensures the correct _unit of
atomicity_ and interaction between parent and child transactional activities.

### 14.3. What triggers the promotion of a local transaction to a distributed transaction with MSDTC in TransactionScope?

**Answer:**  
A transaction is promoted to a distributed (MSDTC-coordinated) transaction when
multiple durable resource managers need to participate—typically:

- Opening multiple connections (to different databases, or the same with
  different connection strings) within the same scope.
- Using two different database providers.
- Making certain ADO.NET provider calls that do not support single-phase commit.

MSDTC must be configured and running on all involved machines. Distributed
transactions incur greater overhead and complexity.

### 14.4. How do you use TransactionScope in asynchronous code, and what mistakes should you avoid?

**Answer:**  
For asynchronous methods using async/await, instantiate TransactionScope with
`TransactionScopeAsyncFlowOption.Enabled`:

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    await SomeAsyncDbOperation();
    scope.Complete();
}
```

If you forget the `AsyncFlowOption`, the transaction will not propagate across
awaits, and you may silently miss transactional protection, or get runtime
exceptions in recent .NET releases.

### 14.5. Describe how exception handling works within a TransactionScope block.

**Answer:**  
If an exception occurs inside the TransactionScope and `Complete()` is not
called, then when the scope is disposed (exits using block), the transaction is
**rolled back automatically**. You should avoid swallowing exceptions silently;
always log exceptions and only call `Complete()` when all work succeeds.

### 14.6. Can you use the same SqlConnection object in different TransactionScopes?

**Answer:**  
No. A SqlConnection participates in the ambient transaction active at the time
it is opened. Reusing the same connection across scopes—especially those not
nested—can lead to `InvalidOperationException` or uncommitted work. Always
manage the connection's lifecycle carefully within the surrounding
TransactionScope block.

### 14.7. How can you avoid unnecessary escalation to distributed transactions?

**Answer:**

- Use a single connection when possible.
- Ensure all connections use identical connection strings.
- Avoid multiple connections unless distributed behavior is required.
- Use promotable single-phase enlistment features where supported. Proper design
  can prevent the performance and complexity costs of MSDTC.

### 14.8. How does TransactionScope integrate with Entity Framework? Compare to using DbContextTransaction.

**Answer:**  
With TransactionScope, you can wrap multiple EF DbContexts, database
connections, or even other resource managers in a single ambient transaction.
`DbContextTransaction`, on the other hand, provides explicit transaction control
for a single context (and database). Use TransactionScope for **cross-context/DB
coordination**; use `DbContextTransaction` for **fine control inside a single
context**. Mixing the two is possible but rarely necessary.

### 14.9. What happens if you call SaveChanges() inside a TransactionScope and Commit is not called?

**Answer:**  
The call to `SaveChanges()` applies the operations to the database, but if
`scope.Complete()` is not called, upon disposal of the scope (end of using), the
transaction is rolled back, and all changes by `SaveChanges()` are reverted.
Hence, `Complete()` acts as the commit action for the entire block.

### 14.10. What are common pitfalls when using TransactionScope with async/await?

**Answer:**

- Not enabling async flow with `TransactionScopeAsyncFlowOption`.
- Using unsupported database providers or platforms.
- Unexpectedly leaking database connections or causing deadlocks if the scope is
  too broad. Test such code extensively in real workloads.

---

## 15. Summary and Recommendations

The .NET `TransactionScope` API is a versatile, high-level abstraction that
**dramatically simplifies transactional programming** in C#, supporting
everything from single-database operations to complex distributed transactions
via MSDTC. It allows developers to focus on business logic without being mired
in complicated transaction management, leveraging ambient context to
automatically enlist eligible resources.

**When implementing TransactionScope:**

- **Understand escalation triggers and try to design transactions to avoid
  distributed coordination unless absolutely necessary**.
- **Always tailor isolation level and timeout based on business needs and
  expected workload**.
- **Favor concise, clear transaction boundaries**, with early disposal and
  explicit commit only on known success.
- **Ensure transactional flow in async code**, and keep abreast of platform and
  provider limitations as technologies evolve.
- **Adopt rigorous logging and monitoring practices**, to provide operational
  oversight and detect issues proactively.

For those preparing for interviews, thorough comprehension of these
topics—including code samples, common pitfalls, and behavioral edge cases—will
demonstrate not only technical mastery but also the practical wisdom that
distinguishes senior professionals.

---

## Appendix: Additional Sample Patterns

### Pattern: Using TransactionScope Across Multiple Data Access Methods

```csharp
using (var scope = new TransactionScope())
{
    // ADO.NET work
    using (var conn = new SqlConnection(connString))
    {
        conn.Open();
        // Execute ADO.NET command
    }

    // Entity Framework work
    using (var context = new MyDbContext())
    {
        context.SomeEntities.Add(new Entity());
        context.SaveChanges();
    }

    // File system or message queue operations (if their ResourceManagers support enlistment)

    scope.Complete();
}
```

### Pattern: Testing Transactional Code with TransactionScope

To test transactional code, **wrap unit test setup in a TransactionScope and
rollback at the end** to ensure no side-effects remain:

```csharp
[TestMethod]
public void TestTransactionalBehavior()
{
    using (var scope = new TransactionScope())
    {
        // test actions
        // assert results
        // Do NOT call Complete(), so all work is rolled back automatically.
    }
}
```

This pattern is popular when **database state pollution must be prevented** in
automated test suites.

---

### EF Core setup (shared code)

```csharp
// DbContext
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders { get; set; }
    public DbSet<AuditLog> AuditLogs { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}

public class Order { public int Id { get; set; } public string Status { get; set; } }
public class AuditLog { public int Id { get; set; } public string Message { get; set; } public DateTime CreatedAt { get; set; } }
```

- Ensure EF Core package references: Microsoft.EntityFrameworkCore and provider
  (e.g., Microsoft.EntityFrameworkCore.SqlServer).
- Register DbContext with DI (AddDbContext) and provide a connection string.

---

### 1. Required propagation (join ambient transaction) — single business operation

- Goal: multiple repository calls join the same transaction so all commits or
  all roll back.
- Approach: use TransactionScope (ambient) or share a single
  DbContext/DbTransaction.

Using a single DbContext (recommended within same logical operation):

```csharp
public class OrderService
{
    private readonly AppDbContext _db;
    public OrderService(AppDbContext db) => _db = db;

    public void ProcessOrder(int orderId)
    {
        // _db is single unit of work; SaveChanges is transactional by default for that DbContext
        var order = _db.Orders.Find(orderId);
        order.Status = "Processed";

        // call repository-like methods that use the same DbContext
        AddAudit($"Order {orderId} processed");

        _db.SaveChanges(); // atomic for this DbContext/connection
    }

    private void AddAudit(string msg)
    {
        _db.AuditLogs.Add(new AuditLog { Message = msg, CreatedAt = DateTime.UtcNow });
    }
}
```

Using TransactionScope to coordinate multiple DbContexts or resource managers:

```csharp
using System.Transactions;

public void ProcessOrderWithScope(int orderId, IServiceProvider sp)
{
    using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
    // create new scope-resolved DbContexts which will enlist in ambient transaction
    var db1 = sp.GetRequiredService<AppDbContext>();
    var db2 = sp.GetRequiredService<AppDbContext>();

    var order = db1.Orders.Find(orderId);
    order.Status = "Processed";
    db1.SaveChanges();

    db2.AuditLogs.Add(new AuditLog { Message = $"Processed {orderId}", CreatedAt = DateTime.UtcNow });
    db2.SaveChanges();

    scope.Complete();
}
```

- Use TransactionScopeAsyncFlowOption.Enabled if awaiting inside the scope.

---

### 2. RequiresNew (independent commit for audit/log)

- Goal: persist audit log even if outer business transaction later rolls back.
- Approach: create a new TransactionScope with RequiresNew or create an
  independent DbContext + explicit DbTransaction and commit it.

TransactionScope RequiresNew example:

```csharp
public void ProcessOrderWithIndependentLog(int orderId, IServiceProvider sp)
{
    using var outer = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);

    var db = sp.GetRequiredService<AppDbContext>();
    var order = db.Orders.Find(orderId);
    order.Status = "Processed";
    db.SaveChanges();

    using var inner = new TransactionScope(TransactionScopeOption.RequiresNew, TransactionScopeAsyncFlowOption.Enabled);
    var logDb = sp.GetRequiredService<AppDbContext>(); // new DbContext instance
    logDb.AuditLogs.Add(new AuditLog { Message = $"Order {orderId} processed", CreatedAt = DateTime.UtcNow });
    logDb.SaveChanges();
    inner.Complete();

    // outer may still fail or roll back; the log remains committed
    outer.Complete();
}
```

Notes:

- Use distinct DbContext instances for independent transactions.
- RequiresNew can escalate resource usage and concurrency contention.

---

### 3. Suppress (run code outside any transaction)

- Goal: call external service or perform work that must not be part of an
  ambient transaction.
- Approach: TransactionScope with Suppress.

Example:

```csharp
public async Task ProcessOrderAndNotifyAsync(int orderId, IServiceProvider sp, HttpClient client)
{
    using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
    var db = sp.GetRequiredService<AppDbContext>();
    // update domain
    var order = await db.Orders.FindAsync(orderId);
    order.Status = "Processed";
    await db.SaveChangesAsync();

    // run notification outside any ambient transaction
    using (var suppressed = new TransactionScope(TransactionScopeOption.Suppress, TransactionScopeAsyncFlowOption.Enabled))
    {
        var resp = await client.PostAsJsonAsync("https://example.com/notify", new { OrderId = orderId });
        suppressed.Complete();
    }

    scope.Complete();
}
```

- Suppressed block sees Transaction.Current == null; external calls won't be
  enlisted.

---

### 4. Savepoints / simulated nested transactions with EF Core

- Goal: perform an inner operation that can be rolled back to a savepoint
  without aborting the outer transaction.
- Approach: use a single DbConnection/DbTransaction and SQL savepoints or EF
  Core 6+ Savepoints API (RelationalDatabaseFacadeExtensions).

Using explicit DbTransaction and raw savepoints:

```csharp
public void OuterWithSavepoint(IServiceProvider sp)
{
    var options = sp.GetRequiredService<DbContextOptions<AppDbContext>>();
    using var db = new AppDbContext(options);
    db.Database.OpenConnection();
    using var tran = db.Database.BeginTransaction(); // DbTransaction

    try
    {
        db.Database.ExecuteSqlRaw("INSERT INTO Orders (Status) VALUES ('A')");
        db.SaveChanges();

        // create savepoint
        db.Database.ExecuteSqlRaw("SAVE TRANSACTION Save1");

        try
        {
            // inner operation that might fail
            db.Database.ExecuteSqlRaw("INSERT INTO SomeTable (...) VALUES (...)");
            db.SaveChanges();
        }
        catch
        {
            // rollback to savepoint
            db.Database.ExecuteSqlRaw("ROLLBACK TRANSACTION Save1");
            // continue without failing entire transaction
        }

        tran.Commit();
    }
    finally
    {
        db.Database.CloseConnection();
    }
}
```

EF Core 6+ supports Savepoints via RelationalTransaction:

```csharp
using var tran = db.Database.BeginTransaction();
tran.CreateSavepoint("sp1");
// ... do work
tran.RollbackToSavepoint("sp1");
tran.ReleaseSavepoint("sp1");
tran.Commit();
```

- Savepoints depend on provider support (SQL Server supports SAVE TRANSACTION;
  others have different SQL or support levels).

---

### 5. Async/await and TransactionScope flow with EF Core

- Goal: keep ambient transaction across awaits when using TransactionScope.
- Key: always use TransactionScopeAsyncFlowOption.Enabled.

Example:

```csharp
public async Task ProcessAsync(int orderId, IServiceProvider sp)
{
    using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
    var db = sp.GetRequiredService<AppDbContext>();
    var order = await db.Orders.FindAsync(orderId);
    order.Status = "Processed";
    await db.SaveChangesAsync();

    await SomeOtherAsyncOperation(); // ambient transaction flows across await

    scope.Complete();
}
```

- Without AsyncFlowOption.Enabled, TransactionScope will not flow across awaits
  and later operations may run outside the intended transaction.

---

### 6. Multiple databases and distributed transactions — prefer outbox

- Problem: updating two different databases in one transaction may trigger
  distributed transaction (MSDTC) escalation.
- EF Core + TransactionScope can escalate when multiple distinct durable
  resource managers are enlisted.
- Recommended pattern: use the Outbox pattern (persist events/commands in same
  DB transaction, then a separate runner publishes reliably).

Outbox sketch:

1. In one transaction (EF Core single DbContext), save entity changes and append
   an Outbox message row.
2. Commit transaction (no DTC).
3. A background worker reads Outbox rows and publishes to other systems (message
   broker), marks them sent.

Simple outbox save example:

```csharp
public async Task ProcessWithOutboxAsync(int orderId, AppDbContext db)
{
    var order = await db.Orders.FindAsync(orderId);
    order.Status = "Processed";

    db.Set<OutboxMessage>().Add(new OutboxMessage {
        Destination = "payment-service",
        Payload = JsonSerializer.Serialize(new { OrderId = orderId }),
        CreatedAt = DateTime.UtcNow
    });

    await db.SaveChangesAsync(); // atomic: order + outbox row
}
```

- A separate worker reads OutboxMessage, sends to messaging system, then marks
  the row as sent.

If you must use distributed transactions, be aware of MSDTC configuration,
network considerations, and performance implications.

---

### Practical recommendations (compact)

- Prefer single DbContext per business operation; SaveChanges provides a
  reliable transaction for that connection.
- Use TransactionScope only when coordinating multiple DbContexts or resource
  managers and enable async flow for async code.
- Use RequiresNew only for isolated commits like audit logs, and use separate
  DbContext instances.
- Use savepoints (DbTransaction or EF Core RelationalTransaction) to simulate
  nested rollbacks.
- Avoid cross-database distributed transactions; prefer outbox/event-driven
  integration for reliability and scalability.

---

## Transaction propagation overview

Transaction propagation describes how transactional context flows across method
calls, threads, services, and resource boundaries. Transaction propagation
determines whether a method joins an existing transaction, creates a new one,
suppresses ambient transactions, or uses a nested savepoint-style approach. In
.NET the most common mechanisms are TransactionScope ambient transactions,
explicit DbTransaction objects, and ORM-managed transactions such as EF Core
Database.BeginTransaction. Understanding propagation is essential for
predictable commits, rollbacks, resource enlistment, and performance.

---

### Transaction primitives in C#

- **TransactionScope ambient transaction**
  - Creates an ambient Transaction accessible via Transaction.Current. Methods
    and libraries that use TransactionScope or enlist resources will
    automatically join this ambient transaction.
- **DbTransaction explicit transaction**
  - Created from a DbConnection via BeginTransaction and passed explicitly to
    commands. No ambient flow unless you propagate the object yourself.
- **EF Core transactions**
  - EF Core uses Database.BeginTransaction to create explicit transactions and
    can also participate in ambient TransactionScope when configured.
- **Distributed transactions**
  - If multiple durable resource managers are enlisted in a single transaction,
    the runtime escalates to a distributed transaction coordinated by MSDTC.
    Avoid unnecessary escalation.

---

### Common propagation modes and their meaning

- **Required**
  - Use the existing ambient transaction if present, otherwise create a new one.
    This is the default behavior for TransactionScope with default options.
- **RequiresNew**
  - Always create a new, independent transaction. The outer transaction
    continues separately; its outcome does not directly rollback or commit the
    inner transaction.
- **Suppress**
  - Execute without an ambient transaction. Any work inside runs
    non-transactionally even if the caller has a transaction.
- **Nested savepoint style**
  - .NET does not provide true nested transactions across all resource managers.
    Some databases support savepoints which ORMs can use to simulate nested
    transactions. This requires explicit support from the provider and
    developer-managed savepoints.

---

### Scenario 1 Single database operation Using TransactionScope

Description  
Single business operation that updates a single SQL database and needs
atomicity.

Code

```csharp
public void UpdateOrder(int orderId)
{
    using (var scope = new TransactionScope())
    {
        using(var conn = new SqlConnection(connString))
        {
            conn.Open();
            var cmd = conn.CreateCommand();
            cmd.CommandText = "UPDATE Orders SET Status = 'Processed' WHERE Id = @id";
            cmd.Parameters.AddWithValue("@id", orderId);
            cmd.ExecuteNonQuery();
        }

        scope.Complete();
    }
}
```

Behavior

- The TransactionScope creates an ambient transaction. The SqlConnection enlists
  automatically. Commit occurs when scope.Complete is called. An exception
  prevents Complete and causes rollback.

---

### Scenario 2 Call chain with Required propagation Joining ambient transaction

Description  
A high-level service opens a TransactionScope and calls repository methods that
should join the same transaction.

Code

```csharp
public void ProcessOrder(int orderId)
{
    using(var scope = new TransactionScope())
    {
        orderRepo.UpdateStatus(orderId);
        paymentService.Charge(orderId);
        scope.Complete();
    }
}

public class OrderRepo
{
    public void UpdateStatus(int orderId)
    {
        using(var conn = new SqlConnection(connString))
        {
            conn.Open();
            var cmd = conn.CreateCommand();
            cmd.CommandText = "UPDATE Orders SET Status = 'Processed' WHERE Id = @id";
            cmd.Parameters.AddWithValue("@id", orderId);
            cmd.ExecuteNonQuery();
        }
    }
}
```

Behavior

- orderRepo.UpdateStatus joins the ambient TransactionScope automatically.
  paymentService.Charge also joins if it relies on the ambient transaction.
  Commit or rollback is decided by the outer scope.

---

### Scenario 3 RequiresNew Creating an inner independent transaction

Description  
A logging operation must persist regardless of the outer transaction outcome.
Use RequiresNew to ensure log commits independently.

Code

```csharp
public void ProcessOrderWithLogging(int orderId)
{
    using(var scope = new TransactionScope())
    {
        orderRepo.UpdateStatus(orderId);

        using(var logScope = new TransactionScope(TransactionScopeOption.RequiresNew))
        {
            auditRepo.WriteLog($"Order {orderId} processed");
            logScope.Complete();
        }

        // If outer scope later fails, the log stays committed
        scope.Complete();
    }
}
```

Behavior

- The inner TransactionScope with RequiresNew creates a separate transaction and
  commits when logScope.Complete is called. The outer transaction can later
  rollback without affecting the committed log.

Caution

- RequiresNew may cause multiple transactions against the same database and can
  increase risk of deadlocks. Use sparingly.

---

### Scenario 4 Suppress Running code outside any transaction

Description  
A non-transactional operation must run even when the caller has an ambient
transaction, for example calling an external service that cannot participate in
a transaction.

Code

```csharp
public void ProcessWithoutEnlistment()
{
    using(var scope = new TransactionScope(TransactionScopeOption.Suppress))
    {
        httpClient.PostAsync(...).GetAwaiter().GetResult();
        scope.Complete();
    }
}
```

Behavior

- The suppressed block runs with Transaction.Current set to null. Any resource
  usage inside will not be enlisted in the outer transaction.

---

### Scenario 5 Nested semantics with savepoints Simulating nested transactions

Description  
Perform an operation that can be rolled back to a savepoint while outer
transaction continues. Use database savepoints or EF Core Savepoints.

Code using ADO.NET savepoints

```csharp
public void OuterOperation()
{
    using(var conn = new SqlConnection(connString))
    {
        conn.Open();
        using(var tran = conn.BeginTransaction())
        {
            var cmd = conn.CreateCommand();
            cmd.Transaction = tran;
            cmd.CommandText = "INSERT INTO A ...";
            cmd.ExecuteNonQuery();

            try
            {
                cmd.CommandText = "SAVE TRANSACTION Save1";
                cmd.ExecuteNonQuery();

                // inner operation that might fail
                cmd.CommandText = "INSERT INTO B ...";
                cmd.ExecuteNonQuery();

            }
            catch
            {
                // rollback to savepoint, outer transaction still active
                cmd.CommandText = "ROLLBACK TRANSACTION Save1";
                cmd.ExecuteNonQuery();
            }

            tran.Commit();
        }
    }
}
```

Behavior

- Savepoints allow rolling back part of the transaction while preserving overall
  transaction. Not all providers support savepoints in the same way.

---

### Scenario 6 Distributed transaction Multiple resource managers

Description  
One transaction must update two different resource managers such as SQL Server
and a MSMQ or a second SQL Server instance. The runtime may escalate to MSDTC.

Code using TransactionScope

```csharp
using(var scope = new TransactionScope())
{
    UpdateSqlServerA();
    UpdateSqlServerB();
    scope.Complete();
}
```

Behavior

- If both connections enlist in the same ambient transaction and the resource
  managers are separate durable resource managers, the DTC coordinator is
  invoked. Distributed transactions have performance and deployment
  implications. Avoid escalation by using single connection, sharding strategy,
  or outbox patterns.

---

### Scenario 7 Async code and TransactionScope Async flow

Rules

- TransactionScope supports async flow when created with
  TransactionScopeAsyncFlowOption.Enabled. Without it TransactionScope may not
  flow across await boundaries causing surprising behavior.

Code

```csharp
public async Task ProcessAsync()
{
    using(var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
    {
        await repository.SaveAsync(entity);
        scope.Complete();
    }
}
```

Behavior

- AsyncFlowOption.Enabled preserves the ambient transaction across
  await/continuations. Omitting it may cause operations after await to run
  without the ambient transaction.

---

### Exceptions Rollback and abort semantics

- If TransactionScope.Complete is not called the transaction is marked for
  rollback.
- Any unhandled exception that leaves the scope without calling Complete
  effectively aborts the transaction.
- For explicit DbTransaction, calling Rollback undoes changes. Calling Commit
  finalizes them. Leaving scope without Commit causes rollback if using an
  abstraction that enforces it.

---

### Best practices for predictable propagation

- Prefer explicit, short-lived transactions at the service boundary that
  represent a single business operation.
- Use TransactionScope Option Required for simple joinable flows.
- Use RequiresNew only when you must commit independent work such as audit logs.
  Minimize its use to avoid complexity.
- Avoid distributed transactions on high-throughput paths. Use outbox or
  event-driven patterns to coordinate between services.
- Be explicit about async flow using TransactionScopeAsyncFlowOption.Enabled
  when using TransactionScope with async/await.
- If you need nested rollback semantics prefer database savepoints or ORM
  features that expose savepoints. Do not rely on nested TransactionScope for
  savepoint semantics.
- Centralize transaction boundaries to high-level application services and keep
  repositories and lower-level components transaction-agnostic or designed to
  accept an explicit transaction.

---

### Quick decision table

| Scenario                                                     | Recommended propagation                                           |
| ------------------------------------------------------------ | ----------------------------------------------------------------- |
| Single DB operation                                          | Required via TransactionScope or explicit DbTransaction           |
| Multiple repository calls within one business action         | Required ambient TransactionScope                                 |
| Independent audit/logging commit regardless of outer outcome | RequiresNew                                                       |
| Call to external service that must not be enlisted           | Suppress                                                          |
| Partial undo inside a bigger transaction                     | Savepoints or provider-specific nested transactions               |
| Mult resource managers across hosts                          | Avoid DTC, use outbox; if unavoidable use distributed transaction |
