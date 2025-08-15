---
title: Domain Driven Design
---

Domain-Driven Design (DDD) is a software development approach that prioritizes
the core business domain and its logic.

The primary advantage of DDD is it ensures the software matches the business
needs closely. With DDD, the codebase becomes more organized around the business
concepts, making it easier to manage and evolve over time. Moreover, DDD
promotes a test-driven environment where domain-specific tests can be written,
ensuring the correctness of the business rules implementation.

DDD might not be the best approach in scenarios where the business domain is
simple or when the project is small.

## Key elements of DDD

- Ubiquitous language: Refers to the idea that we should strive to use the same
  terms in our code as the users use. The idea is that having a common language
  between the delivery team and the actual people will make it easier to model
  the real-world domain and also should improve communication.

- Aggregate: A collection of objects that are managed as a single entity,
  typically referring to real-world concepts. Something that has state,
  identity, a life cycle that will be managed as part of the system. A single
  microservice will handle the life cycle and data storage of one or more
  different types of aggregates.

- Bounded context: An explicit boundary within a business domain that provides
  functionality to the wider system but that also hides complexity. Bounded
  contexts contain one or more aggregates. Some aggregates may be exposed
  outside the bounded context; others may be hidden internally.

## Entities and Value objects

An Entity is defined by its identity, not its attributes. It has a unique
identifier that remains constant even if other attributes change over time. For
example, in an e-commerce system, a customer can be considered as an entity
because the customer's details like name, address may change but the customer ID
remains the same.

On the other hand, a Value Object doesn't have an individual identity. They are
immutable and their equality is based on the values they hold.

Value objects are immutable and validate their state upon creation, ensuring
they're always valid. Entities, on the other hand, can change over time but
maintain consistency through invariants, rules that must always hold true.

This contrasts with traditional procedural or functional paradigms where
validation often occurs separately from data manipulation. In these paradigms,
it's common to see a function or procedure that takes raw data, validates it,
then performs some operation. This separation can lead to issues if validation
is forgotten or not updated when data requirements change.

In DDD, by tying validation closely to the data itself, we ensure that our
domain model remains consistent and correct throughout its lifecycle. It also
promotes encapsulation, as clients don-t need to understand how to validate an
object, they just use it.

## Aggregates and Aggregate Roots

Aggregates are clusters of associated objects treated as a single unit, ensuring
data consistency. They contain one or more entities and value objects, with
rules governing their interactions. The Aggregate Root is the entity within the
aggregate that external objects hold references to. It controls access to its
contained entities, enforcing business rules to maintain consistency. For
example, in an e-commerce system, an Order could be an aggregate root, with
LineItems being part of the aggregate.

## Domain Events

They encapsulate the outcome of an action and provide a record of activities
affecting the system's state.

For instance, consider an online shopping platform. A _ProductPurchased_ event
could be triggered when a customer completes a purchase. This event might then
initiate other processes such as updating inventory or sending a confirmation
email to the customer. The use of Domain Events allows for decoupling between
different parts of the application, enhancing modularity and maintainability.

## Repositories

Repositories provide an abstraction layer between the domain and data mapping
layers, allowing a collection-like interface for accessing domain objects. They
encapsulate the logic required to access data sources, promoting separation of
concerns and cleaner code.

Repositories work with Aggregates, clusters of domain objects treated as a
single unit. The Aggregate Root ensures consistency within its boundary by
controlling access to contained entities. It's the only object that can be
directly retrieved from the repository, ensuring integrity.

For complex queries, DDD suggests Specification pattern where business rules can
be combined and reused.

## Strategic and Tactical Patterns

- Strategic Patterns: Focus on the high-level structure and organization of the
  domain and its alignment with business goals. They deal with bounded contexts,
  context mapping, and identifying core, supporting, and generic subdomains.

- Tactical Patterns: Focus on the detailed implementation within the bounded
  contexts. They deal with entities, value objects, aggregates, repositories,
  factories, and domain services.

## Application Service and Domain Service

- Domain Services: Encapsulate business logic within the domain layer, focusing
  on rules and operations intrinsic to the business domain.

- Application Services: Orchestrate domain logic, manage workflows, and
  coordinate use cases within the application layer, focusing on
  application-level concerns.

---

## Interview Questions

### What is the difference between a Domain Model and an Anemic Domain Model, and why does the latter violate DDD principles?

- **Domain Model** → Encapsulates both **state** and **behavior** that enforce
  invariants. Business rules live _inside_ the model.
- **Anemic Domain Model** → Only holds state (data) while business logic sits in
  services. This breaks **object-oriented encapsulation** and turns entities
  into DTOs.
- Why bad in DDD? Because DDD’s goal is to make the model the central expression
  of the domain; moving logic out means you lose _ubiquitous language_ and _rich
  behavior_.

### Explain Aggregates in DDD. Why do we have an Aggregate Root, and what’s its purpose?

- An **Aggregate** is a cluster of domain objects that must be consistent
  together.
- **Aggregate Root**:

  1. Entry point to the aggregate.
  2. Enforces invariants across the aggregate.
  3. Controls access to inner entities — no direct external references to them.

- **Purpose** → To define a clear **consistency boundary** so transactional
  boundaries and invariants are respected.

### What’s the difference between a Value Object and an Entity in DDD?

- **Entity** → Has an identity that persists over time, even if its attributes
  change.
- **Value Object** → Identity is based solely on attribute values; immutable;
  often used for measurements, money, or dates.
- Key point → Changing a VO’s attribute means creating a new instance.

### Why is Ubiquitous Language critical in DDD?

- Ensures developers, domain experts, testers all speak the _same_ terms.
- Reduces misinterpretation in requirements.
- The language is **embedded in code** (class names, method names), preventing
  “translation loss” between business and code.

### How do you decide between putting a business rule in a Domain Service vs. an Entity?

- Put it **in an Entity** if:

  - The rule depends on the entity’s internal state.
  - It changes the entity’s attributes.

- Put it **in a Domain Service** if:

  - The rule spans multiple aggregates/entities.
  - The logic does not belong naturally to a single entity or value object.

- Domain Services still use _ubiquitous language_.

### How do bounded contexts differ from subdomains?

- **Subdomain** → A logical part of the problem space (core, supporting,
  generic).
- **Bounded Context** → A _solution-space_ boundary where a model is defined and
  consistent.
- A subdomain may map to one or more bounded contexts, but BC is more about
  technical and language boundaries.

### When would you use eventual consistency instead of strong consistency in DDD?

- Use **eventual consistency** when:

  - Aggregate boundaries are crossed.
  - You want to scale and avoid distributed transactions.
  - The business can tolerate temporary inconsistencies.

- Typically achieved with **domain events** and asynchronous processing.

### Why might you avoid having large aggregates in DDD?

- Large aggregates cause:

  - Higher contention in transactions.
  - More locking in databases.
  - Performance bottlenecks.

- DDD encourages _small aggregates_ (often a single entity + a few value
  objects) to optimize transactional performance.

### You have an `Order` aggregate with `OrderLines`. The client asks to allow editing of `OrderLine` prices directly. How would you approach it?

- Revisit invariants → If price change affects order total or business rules,
  logic must live in `Order` aggregate root.
- Don’t expose `OrderLine` directly — add a method to `Order` like
  `ChangeOrderLinePrice(...)` so invariants (discount rules, tax recalculations)
  are enforced.

### You have a `Customer` aggregate and a `CreditLimit` check that involves orders and invoices. Where should you place the logic?

- Likely a **Domain Service**:

  - The rule involves multiple aggregates (`Customer`, `Order`, possibly
    `Invoice`).
  - Encapsulate in `CreditService` using domain terms: `CanPlaceOrder(customer,
orderAmount)`.

### How would you implement a domain event and ensure it’s handled only once in a distributed system?

- Persist the event in an **outbox table** within the same transaction as the
  aggregate change.
- Use an **outbox pattern** to publish asynchronously.
- Include an event ID and deduplication logic in consumers

### How do you handle mapping between persistence models (ORM entities) and domain models in DDD?

- Use **mapping layers** or an ORM mapping profile.
- Avoid leaking EF Core entities directly as domain objects.
- Keep persistence concerns (lazy loading, proxies) out of domain.

Alright — here’s your **senior-level DDD rapid-fire interview cheat sheet**.
I’ve kept each Q\&A short so you can recall them quickly in interviews, but
still with enough detail to sound confident and experienced.

### When should you **not** use DDD?

When the domain is simple, CRUD-only, or rules are minimal. DDD adds overhead;
use it for **complex, evolving business rules**.

### How do tactical patterns relate to strategic patterns in DDD?

Strategic → Big picture (bounded contexts, context maps). Tactical →
Implementation inside a bounded context (aggregates, value objects, domain
events).

### What’s a Context Map?

A diagram showing bounded contexts and their relationships (shared kernel,
customer-supplier, anti-corruption layer, etc.).

### What’s an Anti-Corruption Layer (ACL)?

A translation layer to protect your model from external models’ semantics,
avoiding “model pollution”.

### When to use a Shared Kernel?

Only for a **small, stable** set of concepts used identically in multiple
contexts — otherwise, it increases coupling.

### Rule of thumb for aggregate size?

Keep it small enough to update in a single transaction without locking issues;
often a single root entity + value objects.

### Can aggregates reference each other directly?

No direct references; use IDs. Communicate via domain services or domain events.

### When to merge aggregates?

If business rules require strong consistency between them and they’re always
updated together.

### Difference between Entity invariants and Aggregate invariants?

Entity invariants are internal to that entity. Aggregate invariants must hold
across the whole aggregate.

### Why are Value Objects immutable?

To ensure consistency and thread safety, and to make reasoning about equality
simpler.

### How to model a measurement like `Money`?

As a value object with amount + currency, plus domain logic for operations
(addition, conversion).

### Difference between Domain Service and Application Service?

- Domain Service → Pure domain logic, no infrastructure concerns.
- Application Service → Orchestrates use cases, transactions, security.

### When should a domain service be stateless?

Always. They operate on entities/value objects; state lives in aggregates.

### Difference between domain events and integration events?

Domain events → Inside bounded context, part of the model. Integration events →
Cross-context or cross-system, often a translated version of domain events.

### Best practice for event naming?

Past tense, domain language (e.g., `OrderPlaced`, not `PlaceOrder`).

### Why CQRS fits DDD well?

Separates read/write models; allows different optimization for queries vs.
commands; aligns with aggregate boundaries.

### Does CQRS always require event sourcing?

No. Event sourcing is one possible implementation, but CQRS can be done with a
normal database.

### Why map domain models to persistence models instead of using them directly?

Keeps domain free from ORM artifacts, prevents infrastructure leaking into the
model.

### How do repositories fit in DDD?

They act like in-memory collections for aggregates; return fully built
aggregates, not partial objects.

### How to handle cross-aggregate transactions without 2PC (Two-Phase Commit)?

Use eventual consistency with domain events and compensating actions.

### What’s a Core Domain and why is it important?

The part of the domain with the most competitive advantage; focus most modeling
effort here.

### Can you have multiple models for the same concept in DDD?

Yes — if they live in different bounded contexts with different ubiquitous
languages.

### Why is “Big Ball of Mud” the opposite of DDD?

No clear boundaries, mixed models, no ubiquitous language → makes evolution
costly.
