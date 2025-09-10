---
title: Consistency, Availability, and Partition Tolerance
---

## CAP Theorem Overview and Definition

### Definition

The **CAP theorem**, formulated by Eric Brewer in 2000 and formalized by Gilbert
and Lynch in 2002, asserts that _"in the presence of a network partition, a
distributed system must choose between consistency and availability."_ In other
words, it is impossible for any distributed data system to guarantee all three
properties—**Consistency, Availability, and Partition Tolerance—at the same
time**.

- **Consistency (C):** Every read receives the most recent write or an error,
  i.e., all nodes in the distributed system reflect identical data at any given
  moment.
- **Availability (A):** Every request receives a (non-error) response,
  indicating that the system remains responsive at all times, even during
  failures.
- **Partition Tolerance (P):** The system continues to operate correctly even if
  network failures result in some nodes being unable to communicate with others.

Visually, this can be represented as a triangle—no architecture can sit at the
center; it must live on a side, choosing any two out of three at the cost of the
third.

**Key point for interviews:** Due to the inevitability of network partitions in
distributed systems, CAP theorem often "boils down" to a dilemma between
**Consistency and Availability whenever a partition occurs**.

---

## Key Concepts in CAP Theorem

### Consistency in Distributed Systems

**Consistency** asserts that all nodes (servers) in the system see the same data
at the same time. When data is written to one node, that update must be
immediately propagated to all other nodes before any further read is allowed.
This property ensures **every client perceives the same state of data**,
irrespective of which node it connects to.

- **Strong Consistency (Linearizability):** Every read receives the most recent
  write; behaves as if all operations happen instantaneously on a single, atomic
  node.
- **Eventual Consistency:** The system allows temporary inconsistencies after a
  write; given enough time and no new updates, all nodes will converge to the
  same state.
- **Causal, Sequential, Session Consistency:** Intermediate models relax the
  requirement for absolute ordering; useful for collaborative or interactive
  applications where user experience is prioritized over absolute correctness.

**Notably**, CAP Consistency is about synchronization across replicas, different
from the Consistency in ACID (which is about database integrity).

**Example:** In banking, if account balance changes in one branch, the updated
balance must appear across all branches instantly to prevent double-spending.

---

### Availability in Distributed Systems

**Availability** guarantees that every request (read or write) made to the
system gets a (non-error) response, even if some nodes have failed or become
unreachable. All functioning (non-failing) nodes must return a response in a
reasonable time frame, regardless of the state of other nodes or network
conditions.

**Key features:**

- **Continuous Service:** Users always receive either the requested data or an
  explicit message indicating failure.
- **Redundancy and Fault-Tolerance:** Achieved through replication, load
  balancing, and rapid failover techniques.
- **Design Challenge:** In distributed environments, achieving high availability
  often means sacrificing consistency, especially during partitions.

**Example:** Social media or e-commerce sites must remain available even when
parts of the system are temporarily unreachable; users should be able to post or
shop, albeit potentially seeing slightly outdated information.

---

### Partition Tolerance in Distributed Systems

**Partition Tolerance** is the property that ensures a distributed system
continues to operate even when communication failures break the network into
isolated segments (partitions). Nodes within a partition can communicate with
each other but not with those outside.

- **Why It Matters:** Large, physically dispersed systems are always at risk of
  network failures—be it due to hardware faults, cable cuts, or data center
  outages.
- **System Implication:** Partition tolerance is non-negotiable for real-world
  distributed systems; every cloud, microservices architecture, or global
  service must design for the reality of occasional partitions.

**Example:** When a global application has servers in multiple continents,
undersea cable failure can partition North American and European data centers.
Each region must still serve local users despite being disconnected from the
other.

---

## CAP Combinations and Trade-offs

The CAP theorem states that a system can simultaneously guarantee **only two out
of the three properties** during a network partition. This results in three
classes of distributed systems, each with their trade-offs.

| Combination | Guarantees                         | Sacrifices          | Example Systems                | Typical Use Cases                 |
| ----------- | ---------------------------------- | ------------------- | ------------------------------ | --------------------------------- |
| **CP**      | Consistency + Partition Tolerance  | Availability        | HBase, MongoDB, Spanner, RDBMS | Financial, Reservation, Banking   |
| **AP**      | Availability + Partition Tolerance | Consistency         | Cassandra, DynamoDB, CouchDB   | Social media, DNS, Shopping Carts |
| **CA**      | Consistency + Availability         | Partition Tolerance | Traditional DB, Single-node    | Centralized or local systems      |

**Details for Each:**

- **CP (Consistency + Partition Tolerance):** Sacrifices availability—may reject
  or delay operations during partitions to ensure all data remains current and
  correct. E.g., financial transaction platforms.
- **AP (Availability + Partition Tolerance):** Sacrifices consistency—systems
  remain available during partitions but can return outdated or inconsistent
  data, converging later (eventual consistency). E.g., web-based social
  platforms, content delivery networks (CDNs), DNS.
- **CA (Consistency + Availability):** Sacrifices partition tolerance—not
  practical in distributed reality since network partitions are unavoidable.
  Possible only in single-server or tightly-coupled clusters.

---

## CAP Theorem Trade-offs

| Property Combination | Common Use Case         | Trade-off/Notes                       |
| -------------------- | ----------------------- | ------------------------------------- |
| **CP**               | Banking, inventory      | May refuse requests during partitions |
| **AP**               | Social feeds, web cache | Data may be stale during partition    |
| **CA**               | Single-node RDBMS       | Not viable if partitions occur        |

**Further Analysis**:

- **CP:** Rejecting operations is safer in critical systems (banking, stock
  trading) to guarantee correctness. However, this leads to downtime for
  affected users during partitions, impacting availability.
- **AP:** Ensures user experience and uptime (e.g., posting on Twitter during
  global events) but accepts that some information may lag behind the latest
  changes.
- **CA:** True CA is theoretical in real distributed systems. In real life,
  network partitions will happen, so partition tolerance must be built into
  distributed architectures, relegating CA systems to a single machine or
  cluster in the same rack.

---

## Practical Implications for Distributed System Design

The CAP theorem serves as a **strategic lens for system design**, especially
when defining non-functional requirements in an interview or technical
evaluation.

1. **Partition Tolerance is Mandatory:** Real systems must handle partitions, so
   the true choice is always between Consistency and Availability.
2. **Business Context Determines Trade-off:**
   - _Consistency-first (CP):_ Use in banking, ticketing, inventory, and order
     processing—where stale or conflicting data is unacceptable (e.g., two
     people reserving the same seat).
   - _Availability-first (AP):_ Use in newsfeeds, social apps, and services
     prioritizing continuously uninterrupted access. Slightly out-of-date
     information is tolerable (e.g., profile update, eventually consistent
     shopping cart).
3. **Hybrid and Tunable Models:**
   - Some modern databases (DynamoDB, MongoDB, Cassandra) allow configuration of
     per-operation consistency levels, combining AP and CP modes based on
     feature requirements.
   - Certain workflows (shopping cart checkout vs browsing) are engineered to
     switch behavior depending on business logic stage.

**Code/Workflow Example:**

- In an e-commerce site, adding items to a cart may be AP (always succeed),
  while checking out is CP (must confirm no double-purchase).

---

## Interview Questions

### What is the CAP theorem and why is it important in distributed systems?

The CAP theorem states that in any distributed system, it is impossible to
simultaneously guarantee Consistency, Availability, and Partition Tolerance
during network failures. In the real world, since network partitions are
unavoidable, designers must choose to prioritize either Consistency (delivering
correct and up-to-date data) or Availability (ensuring every request receives a
timely response) whenever a partition occurs.

### Define Consistency as it relates to the CAP theorem.

Consistency means all nodes in the distributed system see the same data at the
same time. A successful write is reflected across all nodes instantly; any
subsequent read returns this latest value, not a stale version.

### What does Availability mean in CAP?

In the CAP theorem, Availability is the guarantee that every request made to a
non-failing node receives a response, even if it may not contain the most
up-to-date data. The system is always responsive, regardless of partitions or
node failures.

### What is Partition Tolerance?

Partition Tolerance refers to the system’s ability to continue operating even
when communication between some nodes is lost due to network or hardware
failures. Nodes in each partition can still serve requests, but may not
synchronize with others until the partition is resolved.

### Why can’t a distributed system guarantee all three CAP properties at once?

Because when a network partition occurs and nodes can't communicate, the system
must either:

- Refuse operations that can't be guaranteed up-to-date (sacrificing
  Availability, choosing CP)
- Continue operations with possibly stale data (sacrificing Consistency,
  choosing AP) Partition tolerance itself can't be sacrificed in practical
  distributed environments, so only two of the three properties can ever be met
  during a network split.

### Give real-world examples of each CAP trade-off combination.

- **CP Example:** Banking database (e.g., HBase, MongoDB strict mode, Spanner) —
  ensures correct up-to-date balance or blocks requests during failures.
- **AP Example:** Social media platform, DNS, Cassandra, or DynamoDB — always
  returns a response, sometimes with older data.
- **CA Example:** Monolithic database server; rare for large-scale systems
  because partitions are inevitable.

### How do systems like Cassandra or DynamoDB address CAP theorem trade-offs?

Cassandra and DynamoDB are AP systems by default, prioritizing availability and
partition tolerance (network issues do not block responses), but they support
tunable consistency. In Cassandra, increasing the replication factor and quorum
settings can shift the balance closer to CP for critical operations. DynamoDB
offers per-operation strong consistency but reverts to eventual consistency
during partitions, ensuring AP.

### What is "Eventual Consistency" and where is it relevant?

Eventual consistency is a relaxed form of consistency where all updates to the
system will, over time and in the absence of new writes, reach all replicas,
resulting in uniform state across nodes eventually. It's typical in AP systems,
allowing high availability at the cost of potential temporary divergences—used
in Amazon S3, DynamoDB (eventual mode), and many social platforms.

### Describe a scenario where you would choose Consistency over Availability.

In a ticket booking application, double-booking a seat is unacceptable. During a
network partition, the booking must be blocked, opting for a CP trade-off. Here,
if a node can't confirm with the others that a seat is truly available, it will
reject the request, ensuring data correctness over system uptime.

### Why is CA not considered realistic in modern distributed systems?

Partition Tolerance is not optional due to unpredictable network failures,
hardware faults, or data center issues. Any real distributed system must be able
to operate amidst temporary communication breakdowns; thus, CA without partition
tolerance is theoretical and impractical beyond tightly coupled or single-server
environments.

### Explain what is meant by "quorum-based" protocols and how they affect CAP trade-offs.

Quorum-based protocols require a majority of nodes to agree on a write or read
before acknowledging the operation. Higher quorums can maintain stronger
consistency (CP behavior), but during partitions, they may block progress and
reduce availability. Lowering quorum thresholds favors availability (AP
behavior) but risks returning stale data.

### Does partition tolerance necessarily mean low availability or consistency?

Not always; modern systems minimize trade-offs using advanced mechanisms (e.g.,
multi-region clusters, faster healing, tunable consistency). However, if a
partition does occur, the system must explicitly sacrifice either consistency or
availability, per CAP.

### What is the PACELC theorem and how does it extend CAP?

PACELC states: If a partition (P) occurs, the system must choose between
Availability (A) and Consistency (C) (CAP). Else (E), when running normally, it
must choose between Latency (L) and Consistency (C). This addresses trade-offs
beyond network partitions, such as responsiveness under normal conditions.

### How are microservices architectures influenced by the CAP theorem?

Microservices, often running across multiple hosts or regions, are subject to
CAP trade-offs for their internal state or when communicating with other
microservices. The database or storage choice behind each microservice often
reflects the microservice's need for consistency (e.g., payments, inventory) vs
availability (e.g., recommendations, analytics).

### When designing a system for a hybrid use case (social platform with chat and banking), how would you apply CAP?

You would engineer CP for sensitive components (e.g., funds transfers,
reservation confirmation), but AP for non-critical paths (e.g., chat presence,
public news feeds). Tunable or hybrid stores (e.g., Cassandra with strong
consistency for certain tables, eventual for others) are preferable.—Discussing
these nuanced CAP choices is highly recommended in interviews to demonstrate
judgment.
