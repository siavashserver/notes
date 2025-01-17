---
title: Micro-Services
---

## What are microservices?

Microservices are a software architectural style where an application is divided
into small, loosely coupled services that can be developed, deployed, and
maintained independently.

## What is Cohesion?

The code that changes together, stays together. Cohesion applies to the
relationship between things inside a boundary.

## What is Coupling?

When services are loosely coupled, a change to one service should not require a
change to another. Coupling describes the relationship between things across a
boundary.

Types of Coupling:

- Domain Coupling: This type of interaction is largely unavoidable. As a general
  rule, domain coupling is considered to be a loose form of coupling. A
  microservice that needs to talk to lots of downstream microservices might
  point to a situation in which too much logic has been centralized.

- Pass-Through Coupling: A situation in which one microservice passes data to
  another microservice purely because the data is needed by some other
  microservice further downstream. In many ways it's one of the most problematic
  forms of implementation coupling, as it implies not only that the caller knows
  that the microservice it is invoking calls yet another microservice. The major
  issue with pass-through coupling is that a change to the required data
  downstream can cause a more significant upstream change.

- Common Coupling: Common coupling occurs when two or more microservices make
  use of a common set of data. A simple and common example of this form of
  coupling would be multiple microservices making use of the same shared
  database, but it could also manifest itself in the use of shared memory or a
  shared filesystem.

- Content Coupling: Content coupling describes a situation in which an upstream
  service reaches into the internals of a downstream service and changes its
  internal state. The most common manifestation of this is an external service
  accessing another microservice's database and changing it directly. The
  differences between content coupling and common coupling are subtle. In both
  cases, two or more microservices are reading and writing to the same set of
  data. With common coupling, you understand that you are making use of a
  shared, external dependency. You know it's not under your control. With
  content coupling, the lines of ownership become less clear, and it becomes
  more difficult for developers to change a system.

## What methods are there to break apart an existing monolith?

- Strangler Fig Pattern: You intercept calls to the existing system. If the call
  to that piece of functionality is implemented in our new microservice
  architecture, it is redirected to the microservice. If the functionality is
  still provided by the monolith, the call is allowed to continue to the
  monolith itself.

- Parallel Run: Running both your monolithic implementation of the functionality
  and the new microservice implementation side by side, serving the same
  requests, and comparing the results.

- Feature Toggle: A feature toggle is a mechanism that allows a feature to be
  switched off or on, or to switch between two different implementations of some
  functionality.

## What is a Service Mesh?

A service mesh, deals very narrowly with communication between microservices
inside your perimeter, the _East-West_ traffic.

## What is an API Gateway?

API gateway sits on the perimeter of your system and deals with _North-South_
traffic. Its primary concerns are managing access from the outside world to your
internal microservices. Here is how it works:

- Routing
- Authentication
- Rate Limiting
- Load Balancing
- Logging and Monitoring
- Request/Response Transformations

## What is a load balancer?

A load balancer distributes network or application traffic across multiple
servers to ensure no single server bears too much load.

Load balancing methods:

- Round Robin: Requests are distributed sequentially among the available
  servers.
- Weighted Round Robin: Servers are assigned different weights based on their
  capacity or performance.
- IP Hash: Uses the client's IP address to determine which server will receive
  the request. This can help in cases where session persistence is required, as
  the same client will always be routed to the same server.
- Least Connections
- Least Response Time
- Least Bandwidth

## What is service discovery?

Service discovery is the process of automatically detecting devices and services
on a computer network.

Service discovery methods:

- DNS (Domain Name System)
- Dynamic Service Registries: Kubernetes etcd, Consul

## What is a service registry?

A service registry is a database of available microservices, their instances,
and their locations.

## What is a service facade?

A service facade is a single entry point for a set of microservices, providing a
unified interface to clients.

## What is a message broker?

A message broker is an intermediary that routes messages between different
services or applications.

## What is an event-driven architecture?

An event-driven architecture is a design pattern where events trigger actions in
the system, promoting loose coupling and scalability.

## What is a CQRS pattern?

Command Query Responsibility Segregation (CQRS) separates read and write
operations for better performance and scalability.

---

## Resiliency patterns

### Circuit Breaker

This pattern provides a fail-fast approach, enabling graceful handling of
unexpected errors without bringing down the entire system.

The Circuit Breaker pattern works in three states:

- Closed: All requests are passed through
- Open: Switches to this state when a number of errors or continuous service
  unavailability is encountered, returning an immediate failure response and
  preventing the issue from cascading across the system
- Half-Open: After a certain amount of time, it allows a limited number of
  requests to test recovery of the service. If the service has recovered and
  handles requests with no errors, returns to the closed state. Otherwise, it
  reverts to the open state, allowing more time for recovery.

### Bulkhead

Isolates failures within a service to prevent them from affecting other parts of
the system, by dividing system resources into isolated pools, so that failures
in one pool don't affect the entire system.

There are two main types of bulkhead isolation:

- Resource-level: This type of isolation manages to allocate resources like
  threads and connection pools across different services, ensuring that a
  contention in one service does not affect others.
- Process-level: This strategy focuses on segregating the services into separate
  processes or containers. If one service goes down, the others continue to
  function without being impacted.

### Timeout and Retry

Manages temporary failures and prevents services from waiting indefinitely by
setting timeouts for service calls and implementing retry logic with exponential
backoff to handle transient errors.

### Rate Limiter

Controls the rate at which a client can make requests, and prevents system
overload.

There are several rate limiting strategies:

- Fixed Window: In this strategy, a fixed number of requests are allowed within
  a specific time window. Once the limit is reached, requests are rejected until
  the next time window.
- Sliding Window: The sliding window approach, also known as the token bucket
  algorithm, works by continuously refilling a bucket of tokens that represent
  the allowed number of requests during a time period. When a request arrives, a
  token is consumed. If the bucket is empty, the request is rejected. This
  method allows for more flexible handling of varying traffic conditions.
- Leaky Bucket: Similar to the token bucket, the leaky bucket algorithm imposes
  rate limits by emptying the bucket at a fixed rate. Incoming requests are
  added to the bucket, and if the bucket overflows, requests are rejected. This
  strategy enforces a consistent processing pace at the service.

### Throttling

Is more system-focused, adjusting the rate of processing requests based on
current system load and capacity, by delaying or dropping requests.

### Fallback

It allows an alternative response, known as a fallback response, to be returned
when a service cannot process a request. Ensures that users still receive
meaningful feedback, even if the primary service cannot provide the desired
result. The Fallback pattern can be combined with other resiliency patterns,
such as the Circuit Breaker and Retry patterns, to further enhance the
availability of microservices-based applications.

### Load Shedding

Load shedding is the process of shedding excess load to maintain system
stability. When a system experiences high traffic or resource demand, it can
prioritize important requests and reject or delay less critical ones, ensuring
that essential services continue operating smoothly.

### Caching

Involves storing frequently accessed data in a temporary storage location,
allowing faster access and reducing load on underlying services. Caching can
improve performance and prevent failures by reducing the number of calls to
slower, external services.

---

## Distributed transactions

### Two-Phase Commits (2PC)

The 2PC is broken into two phases: a _voting phase_ and a _commit phase_.

During the voting phase, a central coordinator asks all services that are going
to be part of the transaction for confirmation as to whether or not some state
change can be made. If any service says that the change cannot take place, the
entire operation aborts.

It's important to highlight that the change does not take effect immediately
after a service indicates that it can make the change. Instead, the service is
guaranteeing that it will be able to make that change at some point in the
future, by lock the record to ensure that other changes cannot take place.

If any services didn't vote in favor of the commit, a rollback message should be
sent to all services to clean up locally. If all services agreed to make the
change, we move to the _commit phase_, where changes are actually made, and
associated locks are released.

2PC can be a quick way to inject huge amounts of _latency_ into your system,
especially if the scope of locking is large, or if the duration of the
transaction is large. It's for this reason two-phase commits are typically used
only for very short-lived operations.

### Saga

Saga is by design an algorithm that can coordinate multiple changes in state,
but avoids the need for locking resources for long periods of time. A saga does
this by modeling the steps involved as discrete activities that can be executed
independently. We can break a single business process into a set of calls that
will be made to collaborating services, this is what constitutes a saga.

A _compensating transaction_ is an operation that undoes a previously committed
transaction. To roll back, we would trigger the compensating transaction for
each step in our saga that has already been committed.

There are two types of recovery in sagas:

- _Backward recovery_ involves reverting the failure and cleaning up afterwards.
  We need to define compensating actions that reverts previously committed
  transactions.

- _Forward recovery_ allows us to pick up from the point where the failure
  occurred and keep processing. For that to work, we need to be able to retry
  transactions, which in turn implies that our system is persisting enough
  information to allow this retry to take place.

There is two types of saga implementations:

- Orchestrated sagas: They use a central coordinator (orchestrator) to define
  the order of execution and to trigger any required compensating action. By its
  nature, this is a somewhat coupled approach. The other issue, which is more
  subtle, is that logic that should otherwise be pushed into the services can
  start to become absorbed in the orchestrator instead.

- Choreographed sagas: It aims to distribute responsibility for the operation of
  the saga among multiple collaborating services. If orchestration is a
  command-and-control approach, choreographed sagas represent a trust-but-verify
  architecture. Choreographed sagas will often make heavy use of events for
  collaboration between services.

---

## Deployment strategies

### Multi-Service

This approach is easy, all of the services are upgraded at the same time. But
since all the services are upgraded at the same time, it is hard to manage and
test dependencies. It’s also hard to rollback safely.

### Blue-Green

There is two identical environments: one is _Staging (Blue)_ and the other is
_Production (Green)_. Once testing is done in the staging environment, user
traffic is switched to the staging environment, and the staging becomes the
production.

### Canary

Upgrades services gradually, each time to a subset of users. It is cheaper than
blue-green deployment and easy to perform rollback. However, since there is no
staging environment, we have to test on production. This process is more
complicated because we need to monitor the canary while gradually migrating more
and more users away from the old version.

### A/B Test

In the A/B test, different versions of services run in production
simultaneously. Each version runs an _experiment_ for a subset of users. A/B
test is a cheap method to test new features in production.
