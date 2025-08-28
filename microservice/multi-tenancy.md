---
title: Multi-Tenancy
---

## Fundamentals of Multi-Tenancy in Microservices

At its core, multi-tenancy enables a single software instance or infrastructure
stack to service multiple distinct customer organizations or user groups
(tenants). Each tenant perceives the application as dedicated to them—even
though underlying resources may be shared.

**Why Multi-Tenancy in Microservices?**

- **Resource Optimization**: Allows optimal utilization of infrastructure by
  pooling hardware, software, and operational overhead.
- **Operational Efficiency**: Reduces deployment, update, and maintenance
  efforts by centralizing what would otherwise be duplicated across
  single-tenant deployments.
- **Customization with Isolation**: Tenants can tailor their experience, data,
  and even security policies, with assurances that their data and operations are
  logically (and sometimes physically) separated from others.

### Defining a Tenant

Within SaaS and microservices contexts, a tenant is typically defined as:

- An organization, such as a customer company, whose users share resources and
  configuration.
- A user group, family, or even an individual in specific B2C contexts.

**The architecture must ensure accurate mapping of users and resources to their
appropriate tenant context at all times.**

---

## Shared vs. Isolated Tenancy Models

| Approach                | Pros                                                | Cons                                     | Use Cases                           |
| ----------------------- | --------------------------------------------------- | ---------------------------------------- | ----------------------------------- |
| Full isolation          | Strongest security/compliance, independent scaling  | Cost, operational overhead               | Regulated industries, large tenants |
| Shared app, separate DB | Balance isolation with cost, independent DB scaling | Slightly more complex than shared schema | Medium/large tenants                |
| Shared app, shared DB   | Cost, easy updates, best for many small tenants     | Highest risk of accidental data leakage  | Large SaaS platforms                |
| Hybrid                  | Flexible, per-tenant adaptation                     | Management complexity, migrations        | Tiered offerings, platform SaaS     |

---

## Hybrid Multi-Tenancy Approaches

Real-world implementations often blend multiple strategies, creating a hybrid
model. For example:

- **Big tenants** may be served by isolated databases (or even infrastructure)
  for performance and compliance.
- **Small/medium tenants** may share a schema or database with strong logical
  isolation.

This “best of both worlds” approach enables the platform to offer
price/performance tiers, support organic tenant growth, and accommodate
migrations between models as needed.

---

## Security Implications of Multi-Tenancy

Multi-tenancy magnifies the importance—and complexity—of authentication,
authorization, and tenant isolation:

### Tenant Isolation

- **Data Isolation**: Preventing tenants from accessing others' data is
  mission-critical. Logical isolation (via `tenant_id` fields) is robust but not
  infallible—application vulnerabilities can cause leakage.
- **Resource Contention**: High-usage tenants (noisy neighbors) can degrade
  performance for others if resource controls (CPU, bandwidth, etc.) and fair
  scheduling are not enforced.
- **Compliance**: Regulated tenants (e.g., in healthcare/finance) may require
  strong separation—including physical or logical partitioning at the
  infrastructure level.
- **Insider Threat**: Cloud provider staff or system admins need careful access
  and audit controls to prevent abusing multi-tenant systems.

### Attack Vectors in Shared Environments

- **Cross-tenant data exposure** due to software bugs or misconfigurations.
- **Privilege escalation** if authorization checks are insufficiently tailored
  to the tenant context.
- **Shared authentication tokens** can be leaked and replayed across tenants
  unless tightly scoped.
- **Side-channel attacks** in shared infrastructure or caching layers.

### Best Practices for Security

| Principle                  | Implementation                                                                                         |
| -------------------------- | ------------------------------------------------------------------------------------------------------ |
| Data & Resource Isolation  | Separate databases/schemas; row-level security; encrypted storage per tenant                           |
| Strong Authentication      | Short-lived tokens, MFA, SSO, and federation                                                           |
| Fine-Grained Authorization | Per-tenant RBAC/ABAC models, scoped permissions                                                        |
| Tenant Context Propagation | Propagate authenticated tenant context via secure tokens or headers; enforce checks in _every_ service |
| Centralized Auditing       | Tenant-aware logging, user/tenant operation tracking                                                   |

---

## Scalability Considerations in Multi-Tenant Systems

A principled multi-tenant microservices system must support onboarding new
tenants and scaling existing ones with minimal friction.

**Key Scalability Concerns:**

- **Horizontal scaling**: Shared infrastructures can serve many tenants, but
  must prevent noisy-neighbor effects through quotas and fair scheduling.
- **Onboarding automation**: Provisioning of tenant resources (databases,
  schemas, SSO configs) must be automated for SaaS to scale.
- **Per-tenant isolation**: Ability to move high-value or high-load tenants to
  isolated infrastructure as they grow.
- **Customizable features**: Scalable systems allow per-tenant customization
  (e.g., in risk policies, branding, pricing) without duplicating the codebase.

Example: Real-world SaaS providers (Azure, AWS, Salesforce) enable selective
isolation while optimizing shared infrastructure for smaller tenants, using
automated deployment scripts and service orchestration to spawn, scale, and
manage tenant resources.

---

## User Management and Tenant Isolation

Effectively managing users in a multi-tenant environment requires solutions to
several challenging questions:

- **User-Tenant Mapping**: Map a user to the correct tenant(s). Users may even
  belong to multiple tenants, requiring context-specific session management.
- **Self-Service & Federation**: Tenants often require the ability to manage
  their own users, integrate with their own directory, or configure SSO.
- **User Lifecycle Management**: Automate provisioning, deprovisioning, and role
  management, often delegating authority to trusted tenant admin roles.
- **Delegated Administration**: Allow tenant administrators to configure
  settings, roles, permissions, and branding for their users, but not for other
  tenants.
- **Tenant-aware Logging and Auditing**: All actions and logs must be filtered,
  partitioned, and queryable by tenant for compliance and troubleshooting.

**Diagram: User-Tenant Relationship**

    +----------------------------+
    |         SaaS App           |
    +----------------------------+
            /         \
        Tenant A    Tenant B
        /    \        /   \
     User A1 User A2 User B1 User B2

**Conclusion:** Authorization and authentication must always be tenant-aware;
user lookup, credential storage, and access tokens must be scoping every request
to the correct tenant.

---

## Authentication Strategies for Multi-Tenant Microservices

Authentication verifies _who_ a user or calling service is. In multi-tenant
architectures, authentication must also establish _for which tenant_ the
identity is asserting access.

### Approaches

#### 1. Centralized Authentication (Identity Gateway or API Gateway)

- All authentication occurs at a single gateway (e.g., API Gateway or Auth
  Proxy), separating the perimeter from internal microservices.
- Edge gateway validates credentials (JWT, OAuth2, etc.), parses tenant identity
  claim, and attaches this context to requests for further services.
- Optional: Mutual authentication between gateway and services.

**Advantages:** Simplifies per-service security, single enforcement point for
policies, enables SSO. **Disadvantages:** Vulnerable if gateway compromised; may
impose performance bottleneck; less granular control for per-service policies.

#### 2. Distributed Authentication (Service-Level)

- Each microservice validates access tokens or credentials, parsing tenant
  identity on every call.
- Microservices may use libraries to parse JWTs with embedded tenant, user, and
  role claims.
- Fine-grained risk policies (e.g., per-tenant MFA) can be enforced per-service.

**Advantages:** Greater resilience, services responsible for own security;
enables defense-in-depth. **Disadvantages:** Increased complexity, potential
duplication, token validation logic in every service.

#### 3. Federation and SSO

- Tenants may demand federated authentication using their own SAML, OpenID
  Connect, or Active Directory Federation Services (ADFS).
- SaaS provider acts as a Service Provider, redirecting login to tenant’s chosen
  IdP (Identity Provider) using open standards (SAML, OIDC).
- Tokens returned may include tenant metadata, roles, and user details required
  for access; supports single sign-on across multiple SaaS and tenant apps.

**Federation Diagram:**

```
User -> SaaS App -> Tenant IdP (SAML/OIDC) -> SaaS App with token (tenant_id, roles, user info)
```

**Advantages:** No need for password/cert management by SaaS; tenants control
their own authentication policy. **Disadvantages:** Requires sophisticated
policy configuration; complex token mapping and attributes transformation; risk
of misconfiguration.

#### 4. Token-Based Authentication (JWT/OAuth2)

**Popular standards:**

- **JWT (JSON Web Token):** Self-contained, signed token containing user and
  tenant claims. Passed in Authorization headers across microservices.
- **OAuth2/OIDC:** Used for delegated authorization and SSO, including support
  for machine-to-machine (M2M) and user authentication.

**Multi-Tenancy Implications:**

- Each token must embed both `tenant_id` and user details, with short expiration
  and clear scoping.
- Access tokens are validated at the point of each service or at the perimeter,
  ensuring tenant context cannot be forged or reused.

#### 5. Multi-factor Authentication (MFA) and Custom Policies

- MFA may be enforced globally, per-tenant, or per-role; authentication platform
  (IdP or SaaS) supports configurable risk-based authentication recipes.

#### 6. Impersonation and Delegation

- Some applications (e.g., SaaS troubleshooting support) require admins to
  impersonate/assume user identities for debugging, requiring careful auditing
  and explicit consent.

#### 7. Risk Evaluation and Adaptive Authentication

- Modern IdPs integrate real-time risk scoring—triggering additional checks or
  MFA when anomalous activity is detected per-tenant.

---

## Authorization Strategies in Multi-Tenant Environments

Authorization answers: _Given their identity, what can a user do?_

A robust authorization model for multi-tenant systems must support policy
granularity at both the tenant (organization) and user (individual) levels.

### Models

#### 1. Role-Based Access Control (RBAC):

- Assign users to roles (e.g., tenant admin, standard user, support agent)
  within a tenant.
- Roles determine permitted actions and resource scopes, often encoded as claims
  within access tokens.
- **Multi-tenancy:** Role assignment and management occurs _per tenant_—the same
  user might be an admin in one tenant and a regular user in another.

#### 2. Attribute-Based Access Control (ABAC):

- Authorizes based on dynamic attributes—user department, region, resource tags,
  operational context.
- Policies can express, for instance: “Allow access to all users in the EU
  region to resources tagged as `region: EU` during business hours.”

#### 3. Policy-Based Access Control and Decision Points

- Centralized policy engines (e.g., AWS Verified Permissions, OPA – Open Policy
  Agent) evaluate complex policies outside of application/service code.

**Policy Decision Point (PDP) and Policy Enforcement Point (PEP):**

- **PDP**: Evaluates policies centrally—you POST context and request; PDP
  replies with allow/deny + explanation.
- **PEP**: Implemented at API endpoints/services; intercepts requests, calls PDP
  for decision.

**Pros:** Auditable, repeatable, less error-prone than bespoke logic scattered
across codebase. **Cons:** Potential performance bottleneck unless carefully
designed; risk of over-centralization.

#### 4. Scoping and Propagation of Tenant Context

- Tenant identity is propagated in access tokens (JWT claims) or as metadata
  (e.g., HTTP headers) to every downstream microservice.
- Every service, when acting on a request, _must_ check that the authenticated
  user’s tenant matches the resource’s tenant.

#### 5. Cross-Tenant Access and Delegation

- For advanced scenarios (e.g., super-admin, support, delegated access),
  policies must define—and securely enforce—limited cross-tenant authorizations,
  always with strong auditing and in line with compliance regulations.

#### 6. Application-Level and Database-Level Isolation

- In addition to authorization checks in business logic, services should enforce
  additional row-level or attribute-based isolation in queries (e.g., always
  filtering on `tenant_id`).

---

## Federation and Single Sign-On (SSO)

### Federation Patterns

- _Federation_ allows authentication delegates to external IdPs—users sign in
  via their organization's preferred provider, not via a SaaS-internal user
  store.
- _Single Sign-On (SSO)_ enables users to authenticate once and access all
  apps/services associated with their tenant without further logins.

**Protocols:** SAML, OAuth2, OpenID Connect.

**Benefits:**

- Centralized policy enforcement (MFA, password policies, geo/access controls)
  at the identity provider.
- Seamless experience for users crossing multiple apps/microservices.
- Simplified onboarding/offboarding: disabling at IdP immediately removes access
  to all federated SaaS applications.

**Implementation Concerns:**

- Syncing user/role information between tenant’s IdP and SaaS.
- Mapping federated claims to tenant-specific roles (user-provisioning
  automation, just-in-time user creation).
- Handling tenants using different federation protocols or user stores.

---

## Token Management and Tenant Context Propagation

Passing tenant context—securely and reliably—involves design decisions at both
protocol and implementation levels:

- **JWT Claims:** Access tokens (JWTs) include `tenant_id`, `user_id`, scopes,
  and roles.
- **Header Propagation:** Tenant context can be carried in custom headers
  (`X-Tenant-ID`) for REST/Kafka or as part of message metadata in event-driven
  systems.
- **Scoped Tokens:** Limit tokens with audience restrictions to only intended
  microservices or parts of the architecture.

**Important Considerations:**

- **Token Longevity:** Use short-lived tokens and refresh tokens via secure
  means (e.g., HTTP-only cookies) to reduce exposure if leaked.
- **Token Validation:** Each service must validate the token’s authenticity,
  expiration, and that it matches the expected tenant context for the operation.
- **Token Exchange:** For multi-step workflows, intermediate services may need
  to exchange tokens for new tokens scoped to their own permissions—or perform
  chained delegation in a controlled manner.

---

## Frameworks and Technologies

A broad spectrum of technologies underpin multi-tenant microservices,
particularly for authentication and authorization. Below is a summary of notable
platforms and tools:

| Technology / Service                         | Purpose                                                  | Example Usage Context                                 |
| -------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------- |
| **Microsoft Entra ID**                       | Enterprise identity platform, federation, SSO            | Organizational SaaS, B2B, Azure ecosystem             |
| **AWS Cognito / Verified Permissions / IAM** | Identity and policy engines for AWS workloads            | Serverless/microservices, fine-grained tenant control |
| **Spring Security (Java)**                   | Authentication/authorization toolkit                     | Java/Kotlin microservices using Spring Boot           |
| **Keycloak**                                 | Open-source identity provider supporting SAML/OIDC, RBAC | Secure user/tenant authentication and federation      |
| **Open Policy Agent (OPA)**                  | Centralized policy enforcement engine                    | Declarative access policies for cloud-native apps     |
| **API Gateways**                             | Centralized authentication/proxy point                   | AWS API Gateway, Kong, NGINX reverse proxy            |

### Supporting Libraries

- **JWT libraries** exist in all major languages (Java, Node.js, Go, Python,
  etc.) for token generation and parsing.
- **Service Meshes** (Istio, Linkerd) support secure, per-tenant mTLS,
  tenant-aware routing, and policy enforcement.
- **Configuration**: Tools like Spring Cloud Config and HashiCorp Vault support
  per-tenant configuration and secret management.

---

## Best Practices and Design Patterns

### General Principles

- **Always Pass and Check Tenant Context**: Every request—internal or
  external—must carry authenticated tenant identity, verified at each processing
  step.
- **Least Privilege Authorization**: Role and attribute scopes should grant
  users/systems only the minimum permissions needed for each tenant context.
- **Centralize Sensitive Policy Logic**: Use PDP/PEP design to enforce policies
  consistently, simplify audits, and avoid code drift across services.
- **Automate Onboarding/Offboarding**: Tenant and user lifecycle management must
  be highly automated for scalable, error-resistant SaaS operations.
- **Regular Auditing**: All user and service actions must be auditable,
  tenant-scoped, and retained as per compliance mandates.
- **Continuous Monitoring and Penetration Testing**: To uncover
  misconfigurations and ensure tenant boundaries are never crossed.

### Detailed Pattern: Token-Driven Tenant Authorization Flow

```
User logs in via tenant IdP / SaaS login     ──▶   Token issued (JWT with tenant_id, roles)
                                                      │
                                                      ▼
API Gateway / First Service validates token           ──▶   Attaches tenant context
                                                      │
                                                      ▼
Downstream service validates token, enforces
tenant_id in business logic & data queries
```

**Every service must fail-closed on any token or tenant context anomaly.**

---

## Diagrams

### 1. Multi-Tenant Authentication and Authorization Flow (Conceptual)

```
                     +---------------------+
[User/Client App] -->| Edge Gateway/AuthN  |---+
                     +---------------------+   |
                          |                    |
   (Validates SSO/JWT)    v                    |
                     +---------------------+   |
                     |  Microservice 1     |<--+
                     +---------------------+    \
                          |  ^                  (Tenant token, roles propagated via JWT/headers)
                          v  |
                     +---------------------+
                     |  Microservice 2     |
                     +---------------------+
                          |
                          v
                     [Database/Data API]
```
