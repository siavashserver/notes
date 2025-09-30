---
title: Entity Framework
---

# Transaction Fundamentals in EF Core

At its core, a **database transaction** is a unit of work that is executed in an
all-or-nothing fashion, providing the ACID properties (Atomicity, Consistency,
Isolation, Durability) essential for data integrity in concurrent applications.
EF Core provides built-in facilities for both **implicit** and **explicit**
transaction management, allowing developers to choose the approach that best
fits their needs.

EF Core abstracts transactions differently based on the underlying provider
(e.g., SQL Server, PostgreSQL, SQLite). However, the concepts and API usage are
largely provider-agnostic, except for distributed transactions and advanced
isolation features, which may differ between databases.

---

## Supported Transaction Types in EF Core

### 1. Implicit Transactions (via SaveChanges / SaveChangesAsync)

EF Core's most basic form of transaction handling occurs **implicitly** every
time you call `SaveChanges()` or `SaveChangesAsync()`. By default, **each call**
is wrapped in a transaction to ensure atomicity of the operations that the
context is tracking.

```csharp
// Implicit transaction - EF Core wraps this SaveChanges call in a transaction
context.Add(new Account { Balance = 100 });
context.SaveChanges(); // EF Core starts and commits the transaction automatically
```

Implicit transactions are desirable for simple operations, as they reduce
boilerplate and shield you from the complexities of manual transaction control.
However, if multiple SaveChanges or other database calls must be treated as a
single logical operation, explicit transactions or scoping mechanisms are
needed.

---

### 2. Explicit Local Transactions (`BeginTransaction` & `BeginTransactionAsync`)

For multi-step, batched, or cross-context operations, you often require
**explicit control** using `BeginTransaction` (synchronous) or
`BeginTransactionAsync` (asynchronous). These methods return an
`IDbContextTransaction` instance, which you then manage directly.

```csharp
using (var transaction = context.Database.BeginTransaction())
{
    context.Add(new Account { Balance = 50 });
    context.SaveChanges();

    context.Add(new Account { Balance = 75 });
    context.SaveChanges();

    transaction.Commit();
}
```

**Asynchronous Explicit Transaction Example**:

```csharp
await using (var transaction = await context.Database.BeginTransactionAsync())
{
    context.Add(new Account { Balance = 50 });
    await context.SaveChangesAsync();

    context.Add(new Account { Balance = 75 });
    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
```

Explicit transactions are essential when you require strict atomicity across
multiple SaveChanges, and they offer precise control for rollback and isolation
level management.

---

### 3. External Transactions (TransactionScope)

EF Core supports **ambient transactions** via
`System.Transactions.TransactionScope`, allowing you to define a transactional
boundary that can encompass multiple operations, potentially involving multiple
DbContexts or even non-EF resources.

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    context1.Add(new Account { Balance = 20 });
    context1.SaveChanges();

    context2.Add(new Customer { Name = "John" });
    context2.SaveChanges();

    scope.Complete(); // Commits both contexts within the transaction scope
}
```

**Note:** TransactionScope enlists all connections opened within its scope. Not
all providers or database versions support distributed transactions, and
TransactionScope defaults have changed in .NET Core and later—ensure
`TransactionScopeAsyncFlowOption.Enabled` is used for async workflows.

---

## Transaction Scoping in EF Core

### Understanding Transaction Scopes

A **transaction scope** defines the boundaries within which a group of
operations are committed or rolled back as a single unit. Scoping can be managed
**explicitly** via EF Core APIs or **implicitly** using `TransactionScope`.
Scoping is vital for controlling which operations execute under which
transactional umbrella, coordinating either within a single DbContext or across
multiple DbContexts and even different resources, such as message queues.

---

#### Local Scoping (Single DbContext):

Local scoping uses EF Core’s native APIs (`BeginTransaction`, `Commit`,
`Rollback`). All operations within the `using` block share a single database
connection, participating in the same transaction context.

```csharp
using (var transaction = context.Database.BeginTransaction())
{
    // All operations before Commit() share the same transaction scope
    // ...
    transaction.Commit();
}
```

This approach is ideal for single-DbContext operations where maximum performance
and simplicity are favored.

---

#### Ambient Transaction Scoping (`TransactionScope`):

Ambient scoping allows you to include multiple DbContexts or even other
transactional resources. Under the hood, it enlists all database connections
opened within the scope into the same distributed or local transaction,
depending on the provider and transaction manager support.

```csharp
using (var scope = new TransactionScope(TransactionScopeOption.Required,
    new TransactionOptions { IsolationLevel = IsolationLevel.Serializable },
    TransactionScopeAsyncFlowOption.Enabled))
{
    // Operations spanning multiple DbContexts or resources
    scope.Complete();
}
```

Note that TransactionScope may start as a lightweight local transaction, but
will escalate to a **Distributed Transaction Coordinator (DTC)** transaction if
multiple connections or resources require distributed coordination.

---

## Transaction Propagation in EF Core

### What Is Transaction Propagation?

**Transaction propagation** refers to how transactional contexts flow when
method calls are nested—do inner methods join an existing transaction, or do
they start new transactions (suspending or overriding an existing one)? While
the concept is most thoroughly discussed in frameworks like Java Spring, EF Core
integrates with .NET's ambient transaction model to provide basic propagation
semantics.

---

### Propagation Modes

EF Core by itself does not offer multiple propagation modes out of the box.
Instead, propagation behavior is determined by how you use TransactionScope or
how you manage transaction instances manually:

- **Join Existing Transaction:** By default, if an ambient transaction exists,
  new operations join it.
- **Requires New Transaction:** You can start a new transaction even if one
  exists (using `TransactionScopeOption.RequiresNew`).
- **Suppress/Avoid Transactions:** Operations can run outside any transaction
  scope by using `TransactionScopeOption.Suppress`.

**Example: Propagation in Action**

```csharp
void MethodA()
{
    using (var scope = new TransactionScope())
    {
        MethodB(); // MethodB joins transaction started in MethodA
        scope.Complete();
    }
}

void MethodB()
{
    using (var scope = new TransactionScope(TransactionScopeOption.Required))
    {
        // This will join the outer transaction if present
        // ...
        scope.Complete();
    }
}
```

Developers must take care when mixing explicit transactions and
TransactionScope, as improper nesting can lead to confusing propagation results
or even exceptions.

---

## Nested Transactions and Savepoints

### The Need for Nested Transactions

Some business operations involve multiple logical steps that should each be
atomic, yet also part of a larger transaction. True **nested transactions**
(where each inner commit is durable and isolated) are not fully supported by
most RDBMSs. Instead, EF Core and many databases offer **savepoints**—markers
within a transaction to which the transaction can partially roll back.

---

### Using Savepoints in EF Core

EF Core supports savepoints for partial rollbacks within a transaction. The key
methods are `transaction.CreateSavepoint()` and
`transaction.RollbackToSavepoint()`. Support for savepoints depends on the
database provider (e.g., SQL Server 2022+, PostgreSQL).

```csharp
using (var transaction = context.Database.BeginTransaction())
{
    context.Add(new Account { Balance = 80 });
    context.SaveChanges();

    transaction.CreateSavepoint("BeforeSecondInsert");

    context.Add(new Account { Balance = 120 });
    context.SaveChanges();

    // Something goes wrong—rollback to savepoint
    transaction.RollbackToSavepoint("BeforeSecondInsert");

    transaction.Commit();
}
```

**Explanation:** In the code above, if an error occurs after the second insert,
you can roll back only the operations after the savepoint, rather than the
entire transaction. This feature is invaluable for complex business logic, batch
processing, or partial error recovery scenarios.

---

## Configuring Isolation Levels

### Importance of Isolation Levels

**Isolation levels** dictate how transactions interact with each other's data to
balance consistency and concurrency. Lower isolation levels (such as Read
Committed) allow higher concurrency but risk anomalies like dirty reads or
non-repeatable reads; higher levels (such as Serializable) virtually eliminate
these anomalies at the cost of performance due to locking.

---

### Setting Isolation Levels in EF Core

EF Core allows you to specify the isolation level when beginning a transaction:

```csharp
using (var transaction = context.Database.BeginTransaction(System.Data.IsolationLevel.Serializable))
{
    // Serializable isolation ensures the highest data integrity
    // ...
    transaction.Commit();
}
```

**Available Isolation Levels** include ReadUncommitted, ReadCommitted (default),
RepeatableRead, and Serializable. The specific semantics and support for these
levels depend on your database engine.

Choosing the correct isolation level is crucial. For example, financial
transactions may justify Serializable, while reporting workloads typically use
ReadCommitted for maximum concurrency.

---

## Distributed Transactions and DTC Integration

### Motivation for Distributed Transactions

Distributed transactions, coordinated by the **Microsoft Distributed Transaction
Coordinator (MSDTC)**, are required when a transaction must span multiple
databases or resources (e.g., database and message broker) that cannot be
coordinated through a single connection.

---

### Enabling Distributed Transactions in EF Core

EF Core 6+ supports the use of distributed transactions via TransactionScope,
**provided** your database provider and environment support MSDTC. Under the
hood, TransactionScope may escalate a local transaction to MSDTC if it detects
multiple connections or different resource managers.

```csharp
using (var scope = new TransactionScope())
{
    using (var context1 = new MyDbContext())
    using (var context2 = new MyOtherDbContext())
    {
        context1.DoWork();
        context2.DoOtherWork();
        context1.SaveChanges();
        context2.SaveChanges();
        scope.Complete(); // Commits both if MSDTC is available, else fails
    }
}
```

**Key limitations:**

- Not all database providers support distributed transactions.
- In cloud and containerized environments, MSDTC may not be available or may
  require explicit configuration.
- Escalation incurs performance and reliability costs.

Always prefer single-database or local transaction patterns unless distribution
is absolutely necessary.

---

## Error Handling and Rollback Patterns

### Handling Transaction Failures

Robust error handling is fundamental to effective transaction management. EF
Core’s transaction API provides `Rollback()` and `RollbackAsync()` methods.
Always use try/catch blocks to handle exceptions, ensure rollback on error, and
dispose of the transaction object to free resources.

```csharp
using (var transaction = context.Database.BeginTransaction())
{
    try
    {
        context.Add(new Entity { Name = "Test" });
        context.SaveChanges();

        // Maybe another risky operation
        transaction.Commit();
    }
    catch (Exception ex)
    {
        transaction.Rollback();
        // Log or handle the exception as needed
        throw;
    }
}
```

If the transaction is disposed before being committed, changes are rolled back
automatically.

---

### Best Practices for Transactional Error Handling

- **Always wrap explicit transactions in try/catch/finally (or using) blocks.**
- **Log exceptions and include transaction context in your logs for forensic
  traceability.**
- **For savepoints, handle partial rollbacks gracefully without leaving the data
  in an inconsistent state.**
- **Do not mix SaveChanges without a transaction and other operations that
  expect an explicit transaction—or you may end up with uncoordinated state.**

---

## Managing Transactions Across Multiple DbContexts

### Single vs. Multiple DbContexts

In many microservice or modular architectures, you may need to coordinate
changes across multiple DbContexts, each potentially targeting a different
database. In EF Core, you cannot use a single IDbContextTransaction instance
across contexts; instead, use TransactionScope or manually enlist transactions
on each DbContext’s database connection if the provider supports it.

---

### Example: Coordinating Multiple DbContexts

```csharp
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    using (var contextA = new ContextA())
    using (var contextB = new ContextB())
    {
        contextA.Add(new A { Name = "Alpha" });
        contextB.Add(new B { Name = "Beta" });

        contextA.SaveChanges();
        contextB.SaveChanges();

        scope.Complete(); // Both commits or both roll back
    }
}
```

**Note:** If both DbContexts connect to the same database and provider,
TransactionScope uses a single transaction; otherwise, MSDTC is required.

---

## Transaction Performance: Best Practices

### Key Performance Guidelines

Performance-sensitive applications require careful transaction management to
minimize contention, deadlocks, and latency. The following best practices are
recommended:

- **Keep transactions as short as possible.** Long-running transactions hold
  locks and block other operations.
- **Do not use Serializable isolation by default—opt for ReadCommitted unless
  stricter isolation is absolutely required.**
- **Batch multiple changes within a single SaveChanges when possible, rather
  than many small SaveChanges—increases efficiency and reduces transaction
  overhead.**
- **Avoid unnecessary nesting of transactions or overuse of distributed
  transactions.**
- **Profile and diagnose with query plans and EF Core diagnostics tools to spot
  bottlenecks.**

Advanced performance tuning, such as command batching, query optimization, and
reducing the number of tracked entities within a transaction, can drastically
increase throughput.

---

## Logging and Diagnostics for Transactions

### Monitoring Transactional Operations

EF Core provides powerful logging and diagnostic features to monitor transaction
boundaries, savepoints, and errors. By configuring logging, you can capture key
events such as:

- Transaction started/committed/rolled back
- Savepoint creation and rollback
- SQL command execution
- Errors or deadlocks

---

### Enabling Detailed Logging

Configure logging with your chosen logging framework (e.g.,
Microsoft.Extensions.Logging) to monitor transactional behavior:

```csharp
var optionsBuilder = new DbContextOptionsBuilder<MyDbContext>();
optionsBuilder
    .UseSqlServer(connectionString)
    .LogTo(Console.WriteLine, LogLevel.Information); // Or use ILogger
```

For even deeper insight, implement EF Core interceptors (e.g.,
`IDbTransactionInterceptor`, `ISaveChangesInterceptor`) to capture and act upon
transaction events. EF Core also exposes event IDs and categories specifically
for transaction operations, aiding automated diagnostics and alerting.

---

## Code Snippet Table: Transaction Patterns in EF Core

| Scenario                             | Sample Code (C#)                                                            | Notes                                                                              |
| ------------------------------------ | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| Implicit Transaction                 | `context.SaveChanges();`                                                    | Every SaveChanges call is wrapped in a transaction by default.                     |
| Explicit Transaction                 | `using (var tx = context.Database.BeginTransaction()) { ... tx.Commit(); }` | Manually controls commit/rollback boundaries.                                      |
| Async Explicit Transaction           | `await context.Database.BeginTransactionAsync();`                           | Use with `SaveChangesAsync`.                                                       |
| TransactionScope                     | `using (var scope = new TransactionScope(...)) { ... scope.Complete(); }`   | Enlists all opened connections; supports multiple DbContexts and distributed txns. |
| Savepoints (Nested/Partial Rollback) | `tx.CreateSavepoint("sp"); tx.RollbackToSavepoint("sp");`                   | Enables partial rollback inside an explicit transaction, DB support required.      |
| Isolation Level Configuration        | `context.Database.BeginTransaction(IsolationLevel.Serializable);`           | Sets desired transaction isolation; supported levels vary per provider.            |
| Distributed Transactions (DTC)       | `TransactionScope` with multiple connections                                | Escalates to DTC as needed; heavy/complex, ensure infra support.                   |
| Multiple DbContexts Coordination     | `TransactionScope` wrapping multiple contexts                               | All contexts participate if supported; otherwise, escalation or error.             |
| Error Handling/Rollback              | try/catch with `tx.Rollback();`                                             | Ensures rollback on failure, always wrap explicit txns in error handlers.          |

---

Each of these scenarios is detailed in code snippets and discussions throughout
this report. When coding, choose the appropriate pattern for the complexity,
resource scope, and reliability requirements of your operation.

---

## Common Pitfalls and Recommendations

### Pitfalls

- **Mixing SaveChanges without understanding scope:** Without an explicit
  transaction or scope, each call is its own isolated transaction. Data can end
  up partially committed in complex workflows.
- **Improper use of TransactionScope in async code:** Always set
  `TransactionScopeAsyncFlowOption.Enabled` or unexpected results may occur.
- **Distributed transaction requirements ignored:** Attempting to perform
  distributed transactions without properly configured MSDTC leads to runtime
  exceptions or partial failures.
- **Forgetting to call Commit/Complete:** Explicit transactions don't commit
  automatically—always ensure `Commit()` or `scope.Complete()` is called.
- **Provider and version differences:** Not all databases fully support
  savepoints, distributed transactions, or advanced isolation levels—test and
  read your provider’s docs carefully.

### Recommendations

- **Use implicit transactions for single-operation methods/apps, explicit for
  multistep/unit-of-work patterns.**
- **Encapsulate complex operations inside explicit transactions or
  TransactionScopes, keeping their duration as short as business logic allows.**
- **Prefer local transactions over distributed unless cross-database atomicity
  is essential.**
- **Always implement diagnostics and logging, especially around transaction
  failures, rollbacks, and escalations.**
- **Stay aware of changes in underlying DB features and EF Core version
  upgrades, as transactional behavior and supported APIs evolve frequently.**

---

## Summary and Conclusion

Entity Framework Core offers powerful, flexible transaction management suited to
a wide array of application scenarios. Implicit transactions via `SaveChanges`
provide safe defaults for simple changes. For more complex multi-step logic,
**explicit transactions** (local or ambient via TransactionScope) enable
coordinated, atomic units of work. **Savepoints** unlock partial rollback, while
**isolation level** and **distributed transaction** support allow the developer
to strike the right balance between consistency and performance, even in
enterprise or microservices architectures.

The best practice is to choose the simplest workable transaction pattern; start
with implicit transactions and move toward explicit transaction patterns,
savepoints, or distributed coordination only as complexity necessitates. Careful
attention to error handling, performance, logging, and environmental support
(especially for advanced or distributed transactions) is required for robust,
production-grade applications.

Armed with this understanding and practical code patterns, .NET developers can
leverage EF Core’s transaction infrastructure to ensure their data-driven
applications are resilient, high-performing, and maintainable in the face of
real-world demands.

---

Here’s a concise breakdown of how Entity Framework Core handles transactions,
with focused sample code for each concept.

---

## ✅ Supported Transaction Types in EF Core

EF Core supports three main transaction types:

### 1. **Implicit Transactions**

Automatically wraps `SaveChanges()` in a transaction.

```csharp
using var context = new AppDbContext();
context.Users.Add(new User { Name = "Alice" });
context.SaveChanges(); // Implicit transaction
```

### 2. **Explicit Transactions**

Manually control the transaction scope using `BeginTransaction()`.

```csharp
using var context = new AppDbContext();
using var transaction = context.Database.BeginTransaction();

try {
    context.Users.Add(new User { Name = "Bob" });
    context.SaveChanges();

    context.Orders.Add(new Order { UserId = 1 });
    context.SaveChanges();

    transaction.Commit();
} catch {
    transaction.Rollback();
}
```

### 3. **Ambient Transactions (System.Transactions)**

Uses `TransactionScope` for broader transaction control across multiple contexts
or resources.

```csharp
using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
using var context1 = new AppDbContext();
using var context2 = new AppDbContext();

context1.Users.Add(new User { Name = "Charlie" });
context1.SaveChanges();

context2.Orders.Add(new Order { UserId = 1 });
context2.SaveChanges();

scope.Complete(); // Commits both contexts
```

---

## 📦 Transaction Scoping

Scoping defines the boundaries of a transaction. EF Core supports:

- **Single DbContext scope**: Transactions are scoped to one context.
- **Multiple DbContexts**: Requires `TransactionScope` or shared `DbConnection`.

```csharp
var connection = new SqlConnection(connectionString);
connection.Open();

var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlServer(connection)
    .Options;

using var transaction = connection.BeginTransaction();
using var context1 = new AppDbContext(options);
using var context2 = new AppDbContext(options);

context1.Database.UseTransaction(transaction);
context2.Database.UseTransaction(transaction);

context1.Users.Add(new User { Name = "Dave" });
context2.Orders.Add(new Order { UserId = 1 });

context1.SaveChanges();
context2.SaveChanges();

transaction.Commit();
```

---

## 🔁 Transaction Propagation

EF Core itself doesn’t propagate transactions across contexts automatically. You
must:

- Use `TransactionScope` for ambient propagation.
- Share `DbConnection` and manually assign the transaction.

```csharp
using var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);
using var context1 = new AppDbContext();
using var context2 = new AppDbContext();

context1.Users.Add(new User { Name = "Eve" });
context1.SaveChanges();

context2.Orders.Add(new Order { UserId = 1 });
context2.SaveChanges();

scope.Complete(); // Propagates transaction across contexts
```
