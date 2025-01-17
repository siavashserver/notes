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
