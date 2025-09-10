---
title: Micro-Services
---

## What are microservices?

Microservices are a software architectural style where an application is divided
into small, loosely coupled services that can be developed, deployed, and
maintained independently.

## Service-Oriented Architecture (SOA) vs. Microservice

**SOA** orchestrates reusable, network-accessible components (services) into
business processes, promoting interoperability, standardized contracts, loose
coupling, functional autonomy, and ease of maintenance and composition.

| Aspect            | SOA (Service-Oriented Architecture)                                          | Microservices                                                                         |
| ----------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Philosophy**    | Integrates enterprise-wide services for reuse across multiple applications   | Builds small, independently deployable services around specific business capabilities |
| **Communication** | Often uses an **Enterprise Service Bus (ESB)** with protocols like SOAP, XML | Prefers lightweight protocols like REST/HTTP, gRPC, JSON                              |
| **Governance**    | Centralized governance, shared standards                                     | Decentralized governance, each service can choose its own stack                       |
| **Deployment**    | Services may be large and interdependent                                     | Services are small, autonomous, and independently deployable                          |
| **Scalability**   | Scales at the service level, but often tied to ESB constraints               | Scales at the microservice level, enabling fine-grained scaling                       |
| **Use Case**      | Large enterprises integrating legacy systems                                 | Agile, cloud-native, CI/CD-driven environments                                        |

### Key Differences

1. **Granularity**

   - SOA services are **coarse-grained** (cover broad business functions).
   - Microservices are **fine-grained** (focus on a single capability).

2. **Coupling & Independence**

   - SOA services can still have hidden dependencies via the ESB.
   - Microservices aim for **true loose coupling** — each has its own database
     and lifecycle.

3. **Technology Stack**

   - SOA is tech-agnostic but often tied to enterprise middleware.
   - Microservices embrace polyglot persistence and lightweight frameworks.

4. **Data Management**

   - SOA often uses a **shared data model**.
   - Microservices use **database-per-service** to avoid coupling.

5. **Failure Isolation**
   - SOA failures can cascade through the ESB.
   - Microservices isolate failures better with patterns like circuit breakers.

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

A **service mesh** is an infrastructure layer that manages, secures, and
observes communication between microservices within a distributed system. It
achieves this by offloading common concerns—such as service discovery, traffic
management, encryption, and observability—from application code to the
infrastructure itself. The heart of most service meshes is a **sidecar proxy**
deployment, often using high-performance proxies like Envoy, which intercepts
and manages inbound and outbound service traffic.

### Service mesh key concepts

- **Sidecar Proxy Pattern:** Deploying a proxy alongside each microservice
  instance to transparently intercept all service-to-service network traffic.
- **Control Plane:** Centralized components that configure the proxies,
  distributing policies, certificates, and routing instructions.
- **Data Plane:** The collection of sidecar proxies responsible for the actual
  traffic management, security enforcement, and data collection.
- **Service Discovery:** Dynamic discovery and tracking of services and
  endpoints within the mesh.
- **Traffic Management:** Advanced routing, load balancing, traffic shifting,
  and chaos injection features.
- **Security:** mTLS encryption, certificate management, authentication, and
  authorization policy enforcement.
- **Observability:** Collection of fine-grained metrics, distributed traces, and
  detailed logs for effective monitoring and troubleshooting.

### Service Mesh Benefits in Distributed Systems

- **Consistent Security:** Mutual TLS (mTLS) is enforced transparently for all
  service interactions, ensuring data-in-transit encryption and strong workload
  identity verification across the mesh without requiring code changes.
- **Resilience and Fault Tolerance:** Features like circuit breaking, retries,
  and timeouts improve the stability of applications under stress or during
  partial failures.
- **Observability:** Standardized collection of service-level, proxy-level, and
  control plane metrics, as well as distributed tracing and access logs, enables
  robust monitoring and troubleshooting across all services.
- **Traffic Management:** Fine-grained policies for traffic splitting,
  mirroring, and manipulation—essential for canary deployments, blue/green
  releases, and A/B testing—are enforceable declaratively.
- **Operational Efficiency:** Centralized configuration allows operations teams
  to implement policies and update routing or security rules mesh-wide, rapidly
  and consistently, reducing human error and silos.
- **Governance:** Fine-grained access controls, auditability, and compliance
  features become more achievable across the entire microservice landscape.

### How Service Mesh Standardizes and Secures Inter-Service Traffic

- **Zero-trust Network Security:** mTLS encrypts all mesh traffic, authenticates
  endpoints, and enables workload identity-based access control.
- **Resilient Communication:** Automated retries and failovers, circuit
  breaking, and fine-grained timeouts reduce the blast radius of network or
  service failures.
- **Dynamic Routing and Policy Enforcement:** Application traffic can be shaped
  in real time, supporting advanced deployment patterns such as staged rollouts
  or emergency hotpatches without redeployment.
- **Transparent Layer:** The mesh works without requiring application logic
  changes—the proxies handle networking, security, and observability as a
  platform service.

> This architecture empowers platform engineers to enforce system-wide
> invariants (like "encrypt all traffic" or "never expose internal services
> externally") with minimal risk of developer omission.

### Why Kubernetes is a Natural Fit for Service Mesh

**Kubernetes** is the de facto standard for container orchestration, providing
automated resource allocation, scaling, and lifecycle management across
distributed applications. Its **declarative approach** to service discovery,
dynamic scaling, and networking creates fertile ground for service mesh
integration.

A service mesh complements Kubernetes by adding:

- Detecting and dynamically managing mesh membership with the help of Kubernetes
  native service discovery
- Injecting sidecar proxies automatically using Kubernetes admission controllers
  or webhook integrations
- Leveraging Kubernetes APIs for configuration, secret distribution, and
  lifecycle management
- Coordinating updates, rollbacks, or traffic shifts in synergy with Kubernetes
  deployments, replicasets, and network policies

**The result:** Organizations can introduce advanced communication, security,
and observability features to Kubernetes workloads without modifying service
deployments or application code.

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

## What is API Composer pattern?

The **API Composer** pattern (or API composition) enables the gateway to invoke
multiple microservices and assemble their results, returning a unified response
to the client. This is critical where data is distributed across specialized
services—offering a single, cohesive API endpoint to clients. The API composer
often performs in-memory data joins or aggregation calls via serverless handlers
(like AWS Lambda), orchestrating distributed queries in a manner transparent to
clients. Drawbacks include increased coupling (gateway outage impacts all client
calls), potential bottlenecks, and complexity with many microservices and large
datasets. As microservices increase, API composition can suffer from degraded
system availability and higher operational costs due to increased backend
traffic.

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

### Service discovery methods

**Client-Side Discovery** assigns clients the responsibility to query a
centralized registry (like Consul, Eureka, or Zookeeper) to locate target
service instances. The client itself often must implement load balancing logic,
potentially increasing its complexity but removing a dedicated network hop for
load balancing.

**Server-Side Discovery** employs a proxy or load balancer (e.g., AWS ELB,
NGINX) as an intermediary. Clients communicate with this proxy, which queries
the registry and forwards the request to a healthy instance. This abstracts away
service location details from the client and centralizes load balancing, but can
introduce a new bottleneck and requires robust proxy resilience.

**Self-Registration** has each service register itself with the discovery
registry (such as Eureka or Consul) upon startup and deregister upon shutdown.
This model enables dynamic scaling and high availability, but it is essential
that the registry itself is highly available to prevent a single point of
failure.

**Third-Party Registration** utilizes a dedicated sidecar or process to register
and deregister services independently. This is common where main services cannot
directly interact with the registry, such as legacy systems, and leverages
orchestrators or service mesh proxies for registration duties.

**Kubernetes** offers a network-side hybrid service discovery, employing cluster
DNS and distributed proxying (via `kube-proxy` or eBPF alternatives). It blends
the benefits of client and server-side models, maintaining a logical Kubernetes
"Service" with a stable entry point for ephemeral Pods. DNS-based discovery is
used for clients, while endpoints are efficiently load-balanced across cluster
nodes, decoupling clients from Pod lifecycle complexity.

### Leading Tools

- **Consul**, **etcd**, **Zookeeper**, **Eureka** provide robust distributed,
  highly available registries with features like health checking, key-value
  storage, and platform integrations.

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

## What are System Robustness Attributes?

Robustness—as a component of resilience—is defined by a system’s ability to
**resist**, **detect**, **react to**, and **recover from** adverse conditions.
This includes mechanisms for fault tolerance (hardware, software errors),
environmental (power loss, disasters), security (attacks, tampering), and
capacity (overload, resource exhaustion) resilience. A robust system rapidly
protects and restores critical capabilities under duress, ensuring business
continuity and regulatory compliance.

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

### Back Pressure

Essential in reacting to production speed mismatches in streaming or
message-driven systems. When producers emit events faster than consumers can
process (as seen in web, event, or messaging systems), a lack of back pressure
may cause memory overflow, unhandled exceptions, or total system collapse.
Common strategies are pull-based control, explicit rate limiting, and stream
cancellation.

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

### Three-Phase Commits (3PC)

3PC extends the traditional two-phase commit by introducing a pre-commit
"prepared" state, mitigating the so-called "blocking" problem where commit
coordination failures could indefinitely block participants in two-phase commit.
However, 3PC is seldom used in practice due to its complexity, assumptions about
bounded network delays, and its increased round-trip requirements, which can
increase latency considerably. It is valuable for understanding distributed
transaction protocols and is academically foundational.

### Saga

A **Saga** is a **sequence of local transactions**, each executed by a single
service. Rather than synchronizing distributed state via locks or centralized
commits, a Saga coordinates the **eventual completion** of all local steps, and,
if any step fails, invokes **compensating transactions** to undo previously
completed work, thereby restoring system consistency.

#### Key Concepts

- **Local Transactions:** Each service fulfills only its discrete, local portion
  of the overall business process.
- **Compensating Transactions:** Rollback operations are explicitly designed for
  each step to reverse changes in case of failure.
- **Eventual Consistency:** Rather than strong, immediate consistency, the
  system converges to a correct state over time.
- **Decentralization vs Centralization:** Depending on the coordination
  strategy, Sagas can be managed either by a central orchestrator or by
  peer-to-peer event-driven logic.

#### Designing Compensation Logic

For every local transaction in the workflow, an explicit, idempotent
compensation operation must be defined. The compensation function must succeed
even if invoked multiple times and should leave the system in a consistent
state. For example:

- If payment is charged but inventory reservation fails, a **refund payment**
  compensation is required.
- If an order is canceled after shipping failed, a **cancel shipment** and
  **restock inventory** operation is needed.
- The design must ensure that compensation will not break business invariants or
  introduce data inconsistencies even with intermittent failures or retries.

#### Failure Recovery Types

- **Backward recovery** involves reverting the failure and cleaning up
  afterwards. We need to define compensating actions that reverts previously
  committed transactions.

- **Forward recovery** allows us to pick up from the point where the failure
  occurred and keep processing. For that to work, we need to be able to retry
  transactions, which in turn implies that our system is persisting enough
  information to allow this retry to take place.

#### Centralized Control with a Saga Orchestrator

**Orchestration-based Sagas** employ a **central orchestrator
service**—sometimes called a process manager or coordinator—that dictates the
flow of the entire transaction. This orchestrator issues explicit commands to
participant services and handles the order of operation, status tracking, and
all compensation logic if failures occur.

- **Central Workflow Definition:** All saga logic (steps, transitions, error
  handling) reside in the orchestrator.
- **Command-Response Communication:** The orchestrator controls execution by
  sending requests and awaiting results.
- **Centralized Error Management:** Compensation and retries are handled by the
  orchestrator.
- **Visibility and Observability:** The orchestrator maintains an auditable,
  stepwise record of saga progress.

Consider the onboarding of a new employee:

1. The orchestrator invokes the **User Service** to create an account.
2. Upon success, it triggers the **Payroll Service**.
3. On successful payroll setup, it proceeds to the **Benefits Service**.
4. If any step fails, the orchestrator explicitly calls the compensating
   operations of previous services to reverse changes.

#### Decentralized Event-Driven Control with a Choreography-based Saga

In a **choreography-based Saga**, there is **no central coordinator**. Instead,
each participant service knows when to act by **listening for domain events**
published by other services. When a service completes its task, it emits an
event that the next service (or several services) can interpret as a trigger for
the next step in the transaction.

- **Decentralized Control:** Each service contains its portion of workflow
  logic, subscribing to relevant events.
- **Event-Driven Communication:** Services explicitly emit and consume domain
  events.
- **Autonomous Error Handling:** Services handle their own compensation and
  retry logic.
- **Loose Coupling:** Greater scalability and flexibility, as services have
  minimal dependencies on each other.

Following the same e-commerce order scenario:

1. **Order Service** creates an order and emits an `OrderCreated` event.
2. **Payment Service** listens for `OrderCreated`, processes payment, and emits
   `PaymentCompleted` or `PaymentFailed`.
3. **Inventory Service** listens for `PaymentCompleted`. On failure, it emits
   `InventoryReservationFailed`, which triggers compensation in the previous
   service.
4. **Shipping Service** listens for reservation events and handles shipment
   accordingly.

#### Orchestration vs Choreography Saga

| Aspect                  | Orchestration                                    | Choreography                                       |
| ----------------------- | ------------------------------------------------ | -------------------------------------------------- |
| **Control Flow**        | Centralized (single orchestrator)                | Decentralized (event-driven by each service)       |
| **Workflow Definition** | Explicit and managed in orchestrator             | Implicit, spread across services                   |
| **Error Handling**      | Central, via orchestrator                        | Distributed, each service handles compensation     |
| **Complexity**          | Complexity centralizes in orchestrator           | Complexity is distributed, can grow with scale     |
| **Observability**       | High; state and logs are tracked in orchestrator | Harder; must aggregate events across services      |
| **Coupling**            | Moderately coupled to orchestrator               | Loosely coupled, services are more independent     |
| **Scalability**         | Orchestrator could be a bottleneck               | Scales with services; no single point of failure   |
| **Suitability**         | Best for ordered/complex workflows, audit needs  | Best for simple, linear, or highly decoupled flows |
| **Failure Recovery**    | Orchestrator coordinates retry/compensation      | Each service must listen and compensate as needed  |

**In summary**, orchestration is ideal where process visibility, order
enforcement, and centralized error handling are vital. Choreography excels where
flexibility, scalability, and independence are paramount, but adds challenges in
flow tracking and distributed compensation logic.

---

## Distributed Locking in Microservices

Distributed locking is a foundational concept in the synchronization of
operations across multiple processes in large-scale, distributed environments
such as microservices architectures. Unlike traditional, in-memory locks which
restrict access within a single application instance or server, **distributed
locks** must coordinate access to shared resources across many independent nodes
which may reside on different machines, data centers, or even clouds. This
extension in scope introduces a set of complicated challenges including network
latency, node failures, partial system outages, and the risk of deadlocks or
stale locks due to unforeseen process crashes.

At its core, a distributed lock acts as a mutually exclusive access control
primitive. Only one process (among potentially many) may hold the lock at any
given time, thereby providing the critical safety guarantee for concurrent
operations that manipulate shared data, configuration, or hardware. This mutual
exclusion property is essential for consistency and correctness, preventing
issues such as lost updates, race conditions, or the dreaded **double order
processing** bug in e-commerce systems.

In the context of modern **microservices**, the requirement for distributed
locks is acutely felt whenever different service instances or separate
microservices must modify shared state, perform singleton tasks (like leader
election), or coordinate inter-service workflows. For example, suppose a system
scales a background job processor horizontally by deploying replicas; only one
replica should process a specific job at any moment, and a distributed lock
ensures exclusivity. Or, consider a financial system that permits only one
service instance to reconcile accounts—the absence of proper locking could
trigger inconsistencies, duplicated work, or even financial losses.

> It’s important to note that, as per the **CAP theorem**, distributed locking
> generally trades off availability for consistency: if a lock cannot be
> reliably acquired, the system chooses safety over processing requests that
> might violate invariants.

### Why Distributed Locks Are Needed in Microservices

The shift from monolithic architectures to microservices, which is motivated by
scalability, resilience, and ease of deployment benefits, amplifies certain
operational complexities. Microservices commonly share resources such as:

- Databases and transactional stores
- Caches or message queues
- External APIs that are rate-limited
- Hardware resources (e.g., connected devices)
- Stateful configuration repositories

In a horizontally scaled deployment, these resources can quickly become points
of contention. **Race conditions**, for instance, occur when multiple
microservice instances read, update, and write a shared resource nearly
simultaneously, resulting in data corruption or inconsistent state.
Additionally, workflows may require **singleton behavior** (e.g., one-off
scheduled tasks, leadership responsibilities in cluster orchestration, or state
migration processes), which must be carefully coordinated to avoid redundant or
conflicting actions.

Without the discipline imposed by distributed locks, the following risks emerge:

- **Lost updates**: Two processes read the same value, increment independently,
  and both write back, losing one update.
- **Resource overuse**: Two instances might both trigger an external API call
  that must only occur once, potentially breaching SLAs or quotas.
- **Sequencing errors**: Services might perform a sequence-sensitive update
  (like batch processing) in the wrong order, causing inconsistent downstream
  data.
- **Duplicated or missed work**: In leader election or background scheduling
  roles, the absence of a distributed lock may result in either all nodes acting
  (duplication) or none acting (starvation).

### Compliance and Security Drivers

Adoption of distributed locking can also be motivated by compliance and audit
requirements. For instance, security standards such as **ISO 27001** or **SOC
2** emphasize the controlled and auditable handling of sensitive state changes,
which is only possible through reliable coordination provided by distributed
locks. DevSecOps teams in regulated industries frequently depend on distributed
locking to enforce data integrity and workflow guarantees critical to
compliance.

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
