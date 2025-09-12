---
title: CQRS
---

## CQRS (Command Query Responsibility Segregation) Pattern

### Definition and Principles

**Command Query Responsibility Segregation (CQRS)** is an architectural pattern
that distinguishes between read (query) and write (command) operations within a
software system. Traditional CRUD systems employ a single model for both reading
and updating data. In contrast, CQRS breaks this paradigm by creating separate
models for processing commands (which alter system state) and queries (which
retrieve system state), allowing each to be optimized and scaled independently.

This differentiation is rooted in the Command Query Separation (CQS) principle,
attributed to Bertrand Meyer, which states that every method should either be a
command (modifies state, returns nothing) or a query (returns data, does not
alter state), but never both. CQRS elevates CQS from function design to system
architecture.

### Diagram: Basic CQRS Architecture

```
+------------------+        +-------------------+         +------------------+
|    Application   |        |    Command Side   |         |    Query Side    |
|    (API/UI)      |        | (Write Model)     |         |   (Read Model)   |
+------------------+        +-------------------+         +------------------+
         |                           |                             |
   +-----+-----+               +-----+-----+                 +-----+-----+
   |           |               |           |                 |           |
Send Command   Query Data ->   Command Handler               Query Handler
                                |                             |
                         +------v------+          +-----------v-------+
                         |  Domain/Data |          |  Denormalized    |
                         |  Store       |          |  Read Store      |
                         +--------------+          +------------------+
```

In this architecture, commands are routed to the command model (which typically
validates and enforces business rules), and queries are routed to the query
model, whose sole job is to return data efficiently. Notably, the read and write
data stores may be physically separate or distinct only at the application
layer.

### Typical Use Cases

The **CQRS pattern** excels when system read and write workloads have diverging
scalability, performance, or security requirements. It is employed in domains
where business logic complexity for updates far exceeds the logic for reads, and
in applications that require highly optimized, denormalized views for fast,
diverse queries. Common use cases include:

- **Collaborative domains:** Complex business logic for write operations; e.g.,
  banking or insurance systems.
- **Event-driven systems:** Where actions trigger asynchronous processes and
  projections.
- **Systems with high read-write asymmetry:** For instance, e-commerce or social
  networks where read queries (e.g., search, product listings) vastly outnumber
  state changes.
- **Microservices architectures:** CQRS fits naturally into service boundaries,
  allowing independent scaling and technology choices for reads and writes.

Importantly, CQRS is not typically applied across the entire application. It is
advised to use CQRS in bounded contexts where its benefits outweigh its added
complexity.

### Benefits

- **Independent Optimization:** Read and write models can use different schemas,
  data storage, and even technologies. For example, writes may use a relational
  database while reads leverage a document store or cache.
- **Scalability:** Each side can be scaled independently. The read side is often
  scaled horizontally to handle high query volumes, while the write side focuses
  on consistency and integrity.
- **Performance:** Denormalized read models tailored for specific query needs
  avoid the performance penalties of complex SQL joins and aggregations.
- **Security:** Permissions and visibility can be tailored separately for
  commands and queries, reducing the attack surface area.
- **Flexibility:** The system evolves more easily as business requirements
  change, since modification of one side does not directly impact the other.

### Drawbacks and Challenges

However, **CQRS is not a silver bullet**; recognizing its limitations is
critical, especially for senior-level interviews:

- **Increased Complexity:** Designers must develop, deploy, and maintain two
  codebases, two sets of models, and potentially two databases.
- **Eventual Consistency:** In architectures with asynchronous read model
  updates, queries may not instantly reflect the latest writes, leading to
  complications in user experience and testing.
- **Data Synchronization:** Keeping the read and write stores synchronized (and
  reconciling failures) can be challenging, especially in distributed
  environments.
- **Operational Overhead:** Higher cost in both computational resources and the
  expertise required, as more technologies are involved and more failure points
  exist.
- **Not suited to simple CRUD applications:** For simple domains with minimal
  performance or scaling needs, CQRS is considered overengineering.

### Interview Questions

1. _What is the fundamental principle of CQRS?_

   - CQRS separates the read and write responsibilities of a system into
     distinct models, optimizing each for its unique requirements.

2. _Do the read and write models have to use different data stores?_

   - No, they can share a physical store while maintaining different logical
     models, but using separate stores expands flexibility.

3. _What is eventual consistency and how do you handle it in CQRS?_

   - Eventual consistency means the read model may lag behind the write model
     due to asynchronous updates. Handling strategies include user
     notifications, retry policies, or compensating actions.

4. _How does CQRS relate to Domain-Driven Design (DDD)?_

   - The command model typically enforces domain invariants using DDD
     aggregates, while the query model provides optimized data for views and
     UIs.

5. _In what situations is CQRS not recommended?_
   - When the domain is simple, scaling or performance requirements are low, or
     transaction consistency between read and write operations is strictly
     required.

---

## Event Sourcing Pattern

### Definition

**Event Sourcing** is an architectural pattern in which all state changes to a
business object are captured as a sequence of immutable events, stored in an
append-only event store. Rather than saving the current state of an entity, you
store every state-changing event, and the current state is reconstructed by
replaying these events.

This represents a paradigm shift away from the traditional "state-oriented" or
CRUD approach, offering an auditable, historical record resembling an accounting
ledger where each change is chronologically logged and never deleted or
updated—only new events may be appended.

### Example Diagram: Core Event Sourcing Model

```
 Client Command
       |
       v
Command Handler (validates, applies business logic)
       |
       v
Domain Event(s) generated (e.g., AccountDebited, OrderPlaced)
       |
       v
Append Event(s) to Event Store (immutable log)
       |
       v
Optional: Project/Replay Events => Materialized View or Query Model
```

### Use Cases

_Event Sourcing_ is especially valuable when:

- **Audit trails** are critical. For example, financial systems, banking, or
  healthcare records where "who did what and when" matters.
- **Temporal queries** are common, such as reconstructing the state as of any
  point in the past.
- **Business analytics** require tracking user behavior or system evolution over
  time.
- **Complex workflows** demand loose coupling and asynchronous processing.
- **Event-driven microservices** require robust data interchange, enabling
  eventual consistency and replay of missed or failed actions.

Real-world examples include **bank account ledgers** (every deposit/withdrawal
is an event) and **e-commerce order histories** (item added to cart, item
purchased, item shipped).

### Advantages

- **Full Auditability:** All changes are logged, enabling traceability and
  regulatory compliance.
- **Event Replay:** The system state can be rebuilt at any historical point, or
  for test/debug use.
- **Decoupled Workflows:** Downstream processes can subscribe to events for
  updates or triggering computations.
- **Enhanced Fault Tolerance:** Loss of derived state is not catastrophic if the
  event log is intact.
- **Easy Rollback and Replay:** It’s possible to correct errors by writing
  compensation events, not by mutating state.
- **Flexibility for Projections:** The same event stream can support multiple,
  diverse read models or analytics pipelines.

### Drawbacks and Challenges

- **Complexity in Implementation:** Requires robust event schema/versioning and
  careful handling of event evolution.
- **Eventual Consistency Issues:** Materialized views are updated
  asynchronously; system state may lag behind latest events.
- **Query Performance:** Reconstructing state requires event replay; often means
  using projections/materialized views to mitigate slow reads.
- **Migration Difficulty:** Moving a legacy system to event sourcing can be
  costly and disruptive.
- **Overkill for Simple Cases:** For CRUD-based, low-complexity domains, event
  sourcing adds unjustified complexity.

### Example: Basic Event Sourcing Flow

Suppose you are modeling a bank account. Traditional systems would only store
the current balance. With event sourcing, you would store events such as
"AccountCreated", "MoneyDeposited", and "MoneyWithdrawn". To get the current
balance, you replay all MoneyDeposited and MoneyWithdrawn events and compute the
net sum.

### Interview Questions

1. _What is event sourcing and how is it different from traditional state
   storage?_

   - Event sourcing stores each state change as an event rather than just
     persisting the latest state. State is reconstructed by replaying events.

2. _How do you handle schema changes in event sourcing?_

   - By versioning event types, writing adapters or upconverters, and possibly
     storing transformation logic to interpret or migrate legacy event formats.

3. _What are projections in event sourcing?_

   - Projections are read models built by replaying or subscribing to events,
     often serving as materialized views for queries.

4. _What are the challenges of rebuilding state from an event store containing
   millions of events?_
   - Performance is an issue; this is often mitigated by snapshotting the
     interim state periodically and replaying only newer events thereafter.

---

## Event Store: The Foundation of Event Sourcing

### Definition and Architecture

The **event store** is an append-only database designed specifically for the
needs of event-sourced architectures. Rather than storing only current states,
it records every event that mutates system state, maintaining strict
chronological ordering and immutability. Each entity (or aggregate) will have
its own event stream, and the **event store** enables reading the sequence for
rebuilding the current state at any moment in time.

An event store must provide strong guarantees regarding ordering, atomic writes
(typically per aggregate root), immutability, and querying by event stream.

### Event Store vs. Relational Database

| Feature           | Event Store                                 | Relational Database                     |
| ----------------- | ------------------------------------------- | --------------------------------------- |
| Data Model        | Append-only event streams                   | Tabular, normalized                     |
| Data Mutability   | Immutable events (never updated/deleted)    | Rows can be updated/deleted             |
| Recording History | Full event history                          | Only current state, possibly with audit |
| Reconstruction    | State via event replay                      | State directly held in tables           |
| Query Flexibility | Optimized for event stream/replay/subscribe | Optimized for ad-hoc SQL queries        |
| Use Cases         | Event Sourcing, Audit logs, Streaming       | CRUD applications, reporting            |

Event stores, such as **EventStoreDB**, are purpose-built to guarantee
immutability and ordering at scale, supporting operations like event replay,
subscriptions (for projections or external processes), and snapshots for
efficient state restoration. Conversely, relational databases are optimized for
current state queries and do not inherently preserve the granular history of
state transitions; audit tables can simulate this at the cost of complexity and
eventual data loss on deletes/updates.

### Key Event Store Features

- **Immutability:** Events, once written, are never mutated or deleted.
- **Streaming:** Consumers (projections, microservices) can subscribe to new
  events.
- **Transformation and Projections:** Projections or materialized views can be
  built/rebuilt by replaying event streams.
- **Scalability:** Modern event stores leverage distributed architectures and
  economic storage trends to efficiently store large amounts of immutable
  historical data.

### Practical Tools and Frameworks

**Well-known Event Store technologies:**

- **EventStoreDB:** Dedicated event store, native to event sourcing
  architectures.
- **Kafka:** Often used for event streaming; can act as an event store with
  external coordination for ordering/offset management.
- **Relational Databases (Postgres, SQL Server):** Can be adapted with
  append-only tables/schemas for events, but miss some native event store
  features.
- **NoSQL Databases (MongoDB, Cassandra):** Sometimes used for event streams
  with attention to immutability and ordering.

---

## Materialized Views

### Fundamentals

A **materialized view (MV)** is a database object that physically stores the
results of a query, as opposed to normal views which are virtual constructs that
compute results on demand. Materialized views are periodically refreshed and can
dramatically speed up queries—especially those involving complex aggregations,
joins, or subqueries—by serving precomputed, cached data.

While standard views provide real-time access to current data by executing the
query definition against base tables every time they’re accessed, materialized
views trade some freshness for efficiency by storing data and updating it at
defined intervals or on demand.

### Diagram: Materialized View Refresh Cycle

```
[ Base Tables ] --> [ Query ] --> [ Materialized View (MV) ] --->
                                    |           ^
                                    |           |
                  (periodic/triggered refresh)  |
                                    +-----------+
```

Materialized views can be refreshed:

- **Manually**: via explicit refresh commands (e.g., `REFRESH MATERIALIZED
VIEW`).
- **On a schedule**: using cron jobs, scheduled tasks, or database features.
- **On commit / change triggers**: for some RDBMSs that support real-time or
  incremental refresh.

### Use Cases

Materialized views are invaluable for:

- **Data warehousing and analytics:** Frequent, expensive aggregations and
  joins.
- **Reporting dashboards:** Needing near-real-time summaries without incurring
  hefty query costs.
- **Servicing CQRS read models:** Populating denormalized projections for fast
  read access.
- **Replicating remote (or external) data:** Synchronizing or caching data from
  remote servers for local processing.

### Benefits

_Key pros of materialized views include:_

- **Query Performance:** Queries against materialized views are fast, as results
  are precomputed and stored on disk.
- **Resource Optimization:** Reduce workload on production tables and
  transactional databases—expensive calculations run less frequently.
- **Reduced Latency:** End user queries receive cached results, improving
  responsiveness in UI and API contexts.
- **Flexibility:** Complex queries, aggregations, and views can be supported
  which would be too costly in real time.

### Drawbacks

On the other hand, materialized views have noteworthy trade-offs:

- **Staleness:** Data in materialized views can become outdated between
  refreshes.
- **Additional Storage and Maintenance:** They consume extra disk space and may
  need indexing/tuning.
- **Refresh Overhead:** Depending on size and complexity, refreshing an MV can
  be a heavy operation.
- **Potential Inconsistency:** Unless refreshed transactionally (e.g., on
  commit), MV content can lag behind source tables, potentially causing
  confusion for consumers.

### Interview Questions

1. _Explain the difference between a view and a materialized view._

   - A view is a virtual table recomputed for each query, always up-to-date. A
     materialized view is a physical storage of the query result, which is
     refreshed at intervals and may become stale.

2. _How can you keep a materialized view fresh?_

   - Use scheduled refreshes, triggers, or database-specific features like
     `REFRESH MATERIALIZED VIEW`, and, where available, incremental or
     event-driven refreshes.

3. _Describe a scenario where materialized views are advantageous in a CQRS
   system._

   - Serving denormalized, query-optimized read models for high-throughput APIs
     or analytics dashboards that can't tolerate slow, multi-join SQL queries.

4. _What are potential issues with MVs and how do you mitigate them?_

   - Stale data can be mitigated by tuning refresh intervals or using
     incremental updates. Storage overhead is addressed by regular monitoring
     and cleanup.

---

## Integrated View: How CQRS, Event Sourcing, and Their Relatives Work Together

### Architectural Integration

In modern, cloud-native and microservices systems, these concepts are often
combined to maximize flexibility, scalability, and auditability. A typical
architecture layering these concepts would look like:

```
[ User/API Commands ]             [ User/API Queries ]
           |                               |
     Command Handler                   Query Handler
           |                               |
   [Event Sourcing - Produce Events]   [ Read from Materialized Views ]
           |                               |
     Append to Event Store       <==  Projections/Materialized Views built by:
           |                               |
      Publish/Subscribe system   <== [ Event Store Listener ]
           |
   Downstream Consumers (other services, external systems)
```

- **CQRS** segregates handling and models for commands (writes) and queries
  (reads).
- **Event Sourcing** is leveraged on the write side, making the event log the
  canonical source of truth.
- Events are appended to the **event store** and, via publish/subscribe or
  streaming mechanisms, drive **projections** (read models/materialized views).
- The **materialized views** provide fast, scalable read access to various
  consumers; they can be reconstructed as needed by replaying events.
- Both event store and materialized projections utilize underlying **transaction
  logs** for durability and recovery, although at different layers of
  abstraction.

### Diagram: CQRS + Event Sourcing System (Simplified)

```
 +-------------------+   Commands   +-------------------+
 |   Command/API     |----------->  | Command Handler   |
 +-------------------+              +-------------------+
         |                                 |
         |        Apply command, generate   v
         |            1..n Domain Events +------------------+
         +------------------------------>|   Event Store    |----> (Event Subscriptions)
                                          +------------------+
                                                  |
                                             Event Listener/Projector
                                                  |
                                          +----------------------+
                                          | Materialized View(s) |
                                          +----------------------+
                                                  |
                                         +------------------+
                                         | Query/API Layer  |
                                         +------------------+
```
