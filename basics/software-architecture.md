---
title: Software Architecture
---

## N-Layer Architecture

### Structure

N-layer architecture, also called layered or N-tier architecture, segments a
software system into discrete horizontal layers, each addressing a unique aspect
of application functionality. At its most fundamental, this architecture
typically identifies three or more layers:

- **Presentation Layer (UI):** Responsible for rendering information to the user
  and handling user interaction.
- **Business Logic Layer (BLL):** Encapsulates core business rules,
  decision-making, and workflows.
- **Data Access Layer (DAL):** Handles all direct interactions with persistent
  storage—often a database or an API.

More expansive variants might include additional layers such as Service,
Integration, or Infrastructure layers. The N in "N-layer" refers to the
indefinite count of such divisions based on system complexity and needs.

**Diagram:**

```
[Presentation Layer]
        ↓
[Business Logic Layer]
        ↓
[Data Access Layer]
        ↓
[Database/External Resource]
```

### Communication

Communication in N-layered systems is strictly hierarchical:

- Each layer communicates downward with the next immediate layer.
- Upward communication (i.e., lower layers calling higher ones directly) is
  typically discouraged.
- Exception handling can travel upward (errors propagate to previous caller),
  but control and data flow downward.

### Advantages and Disadvantages

| Advantage                       | Disadvantage                                  |
| ------------------------------- | --------------------------------------------- |
| Clear separation of concerns    | Tightly coupled layers can hinder changes     |
| Modularity for independent dev  | Testing is harder due to coupled dependencies |
| Scalable per-layer              | Performance overhead due to layer traversals  |
| Simple for small teams/projects | Over-abstracted, may impede complex features  |
| Familiar to most devs           | Difficult to independently deploy features    |

N-layer architecture excels in straightforward maintainability and is easy for
teams to learn due to its longstanding industry prevalence. However, tightly
coupled layers and dependency chains can slow major refactorings and increase
brittleness as the application scales or requirements evolve.

---

## Hexagonal Architecture (Ports and Adapters)

### Structure

Hexagonal Architecture, also known as Ports and Adapters, was introduced by
Alistair Cockburn to solve coupling and testability problems found in layered
systems. Its key distinction is the centrality of business logic—the
"core"—which is surrounded, not topped or bottomed, by a series of well-defined
interfaces (ports). This design treats all external agents (databases, UIs,
APIs) symmetrically via adapters.

**Diagram:**

```
+------------------------------------------------------------+
|              Adapters (External Systems)                   |
|      +--------------------------------------------+        |
|      |              Ports (Interfaces)            |        |
|      |      +-------------------------------+     |        |
|      |      |        Application Core       |     |        |
|      |      +-------------------------------+     |        |
|      +--------------------------------------------+        |
+------------------------------------------------------------+
```

Visuals often use a hexagon simply for the drawing's sake and not because there
are exactly six interfaces.

### Communication

Hexagonal's communication model consists of:

- **Inbound Ports:** Define how external agents (UI, API consumers) interact
  with the core; typically, these represent application use cases.
- **Outbound Ports:** Define how the core interacts with external resources
  (databases, external APIs).

**Adapters** implement these ports. They translate calls to/from
protocol-specific details and decouple the core from infrastructural concerns.
For testing, mock adapters can seamlessly stand in for real implementations.

**Data Flow Example:**

1. HTTP Request → HTTP Adapter (Inbound) → Use Case (Inbound Port) → Domain
   Logic.
2. Domain Logic → Repository Abstraction (Outbound Port) → Adapter (e.g.,
   SQL/NoSQL implementation) → Database.

Adapters never communicate directly with each other; their interactions are
always mediated through the ports and the core.

### Advantages and Disadvantages

| Advantage                              | Disadvantage                                            |
| -------------------------------------- | ------------------------------------------------------- |
| Strong separation of concerns          | Higher upfront complexity for small apps                |
| Easily testable core                   | Steep learning curve for some teams                     |
| Flexibility: Replace external systems  | Less tool/framework support in some stacks              |
| Infrastructure is completely swappable | Some strictness can overcomplicate simple cases         |
| Highly modular; supports parallel dev  | Little guidance on “how to organize” inside the hexagon |

This architecture is particularly strong for projects that must be adaptable to
rapid technology changes, require extensive testability, or anticipate frequent
integration with different types of external systems (UIs, APIs, data stores).

---

## Onion Architecture

### Structure

Introduced by Jeffrey Palermo, Onion Architecture is an evolution of the Ports
and Adapters/Hexagonal approach, refining the notion of concentric dependency
rings. The fundamental principle is that dependencies point inward toward the
domain model, ensuring no domain logic "leaks" outward to infrastructure or
application platforms.

Typical layers (from innermost to outermost):

- **Domain Layer (Core):** Entities, value objects, and domain services
  representing core business logic.
- **Application Layer:** Use-cases and application services orchestrating
  business workflows.
- **Infrastructure Layer:** Data access, messaging, and communication with
  external services.
- **Presentation Layer:** APIs, UIs, and delivery technologies.

**Diagram:**

```
[ Presentation/UI Layer ]
         ↓
[ Infrastructure Layer ]
         ↓
[ Application Layer ]
         ↓
[ Domain Model (Core) ]
```

Sometimes depicted as concentric circles ("onion rings"), each ring can depend
only on those inside it; only the outermost layer interacts with external agents
(DB, UI, APIs, Tests, etc.).

### Communication

- Dependencies point inward, adhering to the **Dependency Inversion Principle**:
  inner layers define interfaces, which are implemented by the outer layers. No
  outer service is called directly by an inner layer.
- Application services consume domain logic, and coordinate responses to the
  Presentation layer.
- Data access and other infrastructure implementations are “plugged in” via
  dependency injection, allowing the application to run with mocks or in fully
  isolated test modes.

### Advantages and Disadvantages

| Advantage                            | Disadvantage                        |
| ------------------------------------ | ----------------------------------- |
| Enforces domain integrity            | May introduce extra interfaces      |
| Promotes strong testability          | Increased abstraction/complexity    |
| Infrastructure is easily swappable   | Learning curve for new team members |
| Facilitates Domain-Driven Design     | Potentially verbose (boilerplate)   |
| Highly modular, supports large teams | More ceremony for small/simple apps |

Onion Architecture is ideal for complex, business-critical applications where
protecting the domain and ensuring a long-lived system is a core requirement.
There is also a strong synergy with Domain-Driven Design (DDD) practices.

---

## Clean Architecture

### Structure

Clean Architecture, popularized by Robert C. Martin ("Uncle Bob"), distills
patterns seen in Hexagonal, Onion, and others into a systematic set of
concentric circles of abstraction with the **Dependency Rule** at its heart:
source code dependencies must only point inward.

**Layers (from inside out):**

1. **Entities:** Business-wide, core rules.
2. **Use Cases (or Application):** Application-specific business rules and
   workflows.
3. **Interface Adapters:** Controllers, gateways, presenters. Responsible for
   adapting data to/from the use case and interacting with frameworks.
4. **Frameworks & Drivers:** Databases, UI frameworks, message buses, etc.

**Diagram:**

```
+----------------------+
| Frameworks & Drivers |
+----------------------+
|  Interface Adapters  |
+----------------------+
|      Use Cases       |
+----------------------+
|      Entities        |
+----------------------+
```

Each layer can only depend on layers inside it—never the other way around. This
holds for both code and data structures.

### Communication

- Outer layers (Frameworks, Adapters) communicate inward to Use Cases and
  Entities through explicit interfaces defined by the inner layers.
- _Control flow_ can cross from Controller to Use Case to Presenter, but
  compile-time dependencies always point inward.
- Data passed across boundaries should be simple, technology-agnostic structures
  (DTOs or records), not infrastructural models or ORM entities.
- Infrastructure and frameworks are "plugged-in" at app wiring time, making the
  business logic entirely agnostic of persistence or UI details.

### Advantages and Disadvantages

| Advantage                             | Disadvantage                                |
| ------------------------------------- | ------------------------------------------- |
| Absolute separation of business logic | Initial development overhead                |
| Highly maintainable and testable      | Requires discipline to avoid "leaky" layers |
| Framework independent, portability    | Can be “overkill” for simple CRUD projects  |
| Explicit dependency rules             | Potential for over-abstraction              |
| Facilitates agile adaptation          | Added complexity for onboarding             |

Clean Architecture is considered the gold standard for mission-critical,
long-lived systems where business rules must be insulated from technical churn.

---

## Vertical Slice Architecture

### Structure

Vertical Slice Architecture (VSA) offers a fundamentally different organizing
principle: Instead of grouping code by technical layer (UI, BLL, DAL), the
system is split into "vertical slices"—each representing a self-contained
feature. All code relating to a single feature—from request validation to data
persistence—resides together, often grouped within a “feature folder” or module.

Each slice typically includes:

- Request/Command (DTO) models
- Handlers (application/service logic and domain orchestration)
- Validators
- Data access logic (repository or DbContext)
- Endpoints (controllers, API handlers)
- Feature-specific tests

**Diagram:**

```
[Feature: CreateOrder]
|
|-- Controller/Endpoint
|-- Command/Query Request
|-- Handler
|-- Validator
|-- Domain Logic (as needed)
|-- Repository/Data
|-- Response DTO

[Feature: CancelOrder]
(same structure)
```

Beneath the "feature slices" may reside shared infrastructure (ORMS, DBContext,
queue clients), but all logic for a use case is found within a single vertical
folder.

### Communication

- Each feature slice is responsible for orchestrating its full request/response
  pathway (UI→Business Logic→Persistence).
- Code is called vertically through handler/request/validator/data, often using
  libraries like MediatR to decouple concerns within each slice.
- Cross-feature communication should be minimized to maintain modularity; shared
  logic is factorized as utilities or services as needed.

**Flow Example:** HTTP POST `/orders` → CreateOrderController →
CreateOrderCommand/Request → CreateOrderHandler → OrderRepository →
DbContext.Save()

### C. Advantages and Disadvantages

| Advantage                                  | Disadvantage                                |
| ------------------------------------------ | ------------------------------------------- |
| High locality: All feature code co-located | Risk of code duplication                    |
| Easy to reason about changes               | Can reduce consistency across slices        |
| Fast onboarding for new devs               | Feature interaction/coordination complexity |
| Naturally aligns with business goals       | Overhead for factoring shared logic         |
| Facilitates parallel development           | Risk of knowledge silos per feature         |

Ideal for feature-driven teams, rapid prototyping, and systems where independent
feature scaling is desired. Large-scale systems often hybridize VSA with
elements from Clean/Onion/Hexagonal patterns for deeper infrastructural
separation.

---

## Comparison Table

| Architecture   | Structure Clarity | Testability | Framework Independence | Scalability   | Complexity | Flexibility | Dependency Direction    | Best For                                              |
| -------------- | ----------------- | ----------- | ---------------------- | ------------- | ---------- | ----------- | ----------------------- | ----------------------------------------------------- |
| N-layer        | Moderate/Familiar | Moderate    | Low                    | Moderate      | Low        | Low         | Outward (top → bottom)  | Legacy systems, small/medium, conservative IT         |
| Hexagonal      | High              | High        | High                   | High          | Moderate   | High        | Inward (core-centric)   | Integrations, agile systems, test-first               |
| Onion          | High              | High        | High                   | High          | Moderate   | High        | Inward (domain-centric) | DDD, core business with rich logic                    |
| Clean          | Very High         | Very High   | Very High              | High          | High       | Very High   | Strictly inward         | Enterprise, complex, long-lived apps                  |
| Vertical Slice | High for features | High        | High                   | Moderate-High | Moderate   | High        | Contextual (per slice)  | Modern enterprise, fast iteration, parallel dev teams |

---

## Diagrams

### N-layer Architecture

```
[User Interface / Presentation Layer]
                 ↓
        [Business Logic Layer]
                 ↓
        [Data Access Layer]
                 ↓
           [Database]
```

---

### Hexagonal Architecture

```
+---------------------------------------+
|         [ HTTP Adapter ]              |
|         [ UI Adapter   ]              |
|         [ Testing Adapter ]           |
|                 |                     |
|             [ Port ]                  |
|                 |                     |
|        +---------------------+        |
|        |  Application Core   |        |
|        +---------------------+        |
|                 |                     |
|          [Repository Port]            |
|         /          |        \         |
|  [Repository] [Third-Party] [Queue]   |
+---------------------------------------+
```

---

### Onion Architecture

```
+-----------------------+
|   Infrastructure      |
+-----------------------+
|   Application Layer   |
+-----------------------+
|   Domain Model (Core) |
+-----------------------+
```

---

### Clean Architecture

```
+---------------------+
| Frameworks/Drivers  | (DB, UI, Messaging, etc.)
+---------------------+
| Interface Adapters  | (Controllers, Presenters, Repos)
+---------------------+
| Use Cases           | (Application business logic)
+---------------------+
| Entities            | (Enterprise/domain rules)
+---------------------+
```

---

### Vertical Slice Architecture

```
[Feature: CreateOrder]
 ├ Controller/Endpoint
 ├ Request/Handler
 ├ Validation
 ├ Data Access
 └ Response DTO

[Feature: CancelOrder]
 (identical structure, isolated)
```

---

## Interview Questions

### Compare N-layer, Hexagonal, Onion, Clean, and Vertical Slice architectures. When would you choose each?

N-layer is the classic approach organizing systems by technical
responsibilities—presentation, logic, data, etc.—best when modularity and team
separation by specialty are desired, but can lead to tight coupling and testing
difficulties. Hexagonal, Onion, and Clean architectures invert dependencies:
business logic is placed at the core, with all dependencies flowing inward,
maximizing testability, flexibility, and separation of concerns. Clean
Architecture, specifically, codifies layers (Entities, Use Cases, Adapters,
Frameworks) and emphasizes the Dependency Rule, making it particularly strong
for large, long-lived codebases needing high resilience to change. Onion is
similar, especially suited for Domain-Driven Design applications. Hexagonal is
favored for systems interacting with many types of external actors or
integrations. Vertical Slice eschews organizing by technical layers in favor of
feature-based modules, promoting high cohesion and fast delivery of business
requirements, but can introduce duplication.

### How does Clean Architecture enforce the Dependency Rule? Why is this important?

Clean Architecture’s Dependency Rule dictates that source code dependencies can
only point inward, towards high-level policies and abstractions, not outward to
implementation details. Outer layers, such as frameworks and UI, depend on inner
layers (use cases, entities), never vice versa. This ensures the business logic
is insulated from churn in technologies (databases, frameworks, UI drivers) and
enables robust testability and longevity of core logic. The Dependency Inversion
Principle, facilitated by interfaces, enables substitution of infrastructure
without modification to the core and supports highly maintainable systems.

### How do ports and adapters enable testability in Hexagonal Architecture?

Ports define the boundary interfaces that core business logic interacts with,
while adapters implement those interfaces for specific infrastructural concerns
(e.g., a port for data access might have adapters for SQL, NoSQL, or unit
testing). Because core logic never depends directly on implementation, tests can
provide mock adapters, allowing isolated testing of business rules without
needing databases, external APIs, or message queues, reducing time and cost of
testing and increasing coverage.

### Describe the typical folder/project layout for Clean Architecture in a large .NET or Java system.

A common folder structure might include:

- `/Domain` (Entities, core rules, value objects, perhaps domain events)
- `/Application` (Use cases, interfaces for data access, orchestration logic)
- `/Infrastructure` (Repository implementations, data access, framework
  adapters)
- `/WebAPI` or `/UI` (Controllers, presenters, ViewModels, UI-specific
  adaptation) This layout clarifies the dependency direction, makes business
  logic easily identifiable and unit-testable, and ensures infrastructure is
  replaceable with minimal impact to the rest of the system.

### When and why would you choose Vertical Slice Architecture over Clean or Onion Architecture?

Vertical Slice Architecture is especially appealing in rapidly changing domains,
greenfield projects with strong feature or user-story alignment, or when teams
are structured around delivering features independently. It reduces
cross-feature coupling, accelerates onboarding, and allows for easy parallel
work. However, as the codebase grows, careful management of shared logic and
conventions is necessary to avoid fragmentation and inconsistency. It is less
suited to domains where shared, elaborate business logic—or rigorous
layering—dominates the problem space.

### How do you integrate legacy systems with Clean or Hexagonal Architecture in a modernization project?

Legacy systems are commonly wrapped as adapters that implement the relevant
ports/interfaces exposed by the architecture’s core. Over time, functionality
can be migrated slice-by-slice by refactoring user stories or features to run
through the new architecture, often using the "strangler Fig" pattern. This
enables gradual deprecation, continuous delivery, and risk mitigation as the new
system progressively replaces the old.

### What are common pitfalls when implementing Onion or Clean Architecture?

- Over-abstraction: Unnecessary interfaces or layers when not required.
- Leaky Dependencies: Accidental references from inner to outer layers.
- Anemic Domain Model: Storing logic in services/application, not in domain
  objects.
- Ignoring business-driven boundaries: Forcing technology-shaped layers over
  natural business modules.
- Premature optimization for flexibility not justified by the domain/scenario.
- Not aligning boundaries with true business capabilities or subdomains.

### How does dependency injection support Clean, Hexagonal, and Onion Architectures?

Dependency injection is the mechanism by which infrastructure components (repo
implementations, API clients, etc.) are supplied to the application core at
runtime, without the core explicitly referencing their concrete types. This
upholds the Dependency Rule and enables plug-in infrastructure, easy replacement
for testing, and strong decoupling from external systems.

### How does Clean Architecture differ from Onion and Hexagonal in its layering?

Clean Architecture formalizes entity-based and use-case-based inner layers, and
requires transformation at each layer boundary (through DTOs, models, etc.),
whereas Onion typically focuses on domain-centric concentric rings with less
explicit distinction between use cases and adapters. Hexagonal focuses on ports
and adapters rather than explicit concentric abstraction layers, which can make
inner structure more flexible but less prescriptive.

### How does Vertical Slice Architecture help align software development with business goals?

By organizing code by feature, every vertical slice maps directly to a business
requirement, allowing much clearer traceability, faster delivery of value, and
easier prioritization and parallelization. Test coverage and feature
completeness naturally follow business terms, and onboarding on specific
business features is accelerated as all code is locally contained.
