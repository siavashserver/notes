---
title: KeyCloak
---

## Keycloak Overview and Definition

Keycloak is an enterprise-grade open-source identity and access management
system designed to provide secure authentication and authorization for
applications and services. Operating as a standalone server, Keycloak acts
primarily as an Identity Provider (IdP), centralizing user identity management,
credential storage, and access policy enforcement. Rather than building
authentication and authorization logic into each individual application,
organizations can offload these responsibilities to Keycloak, simplifying
security architecture, reducing the technical burden on development teams, and
achieving consistency in security controls.

Key points defining Keycloak include:

- **Centralized SSO and IAM**: Users authenticate once and access all permitted
  applications (Single Sign-On). Keycloak manages identities and access policies
  centrally.
- **Open Standards Compliance**: Full support for OAuth 2.0, OpenID Connect
  (OIDC), and SAML 2.0 for interoperability across a range of applications and
  platforms.
- **Extensible Architecture**: Extensive Service Provider Interface (SPI)
  infrastructure enables custom extensions, protocol mappers, authenticators,
  and advanced integrations.
- **Out-of-Box Features**: MFA, social login, user federation (LDAP, AD),
  identity brokering, fine-grained RBAC, customizable authentication flows, and
  more.
- **Open Source and Community-Driven**: Vendor-neutral under the CNCF with
  community governance, avoiding vendor lock-in and supporting
  enterprise-quality open development.

---

## Keycloak Core Features

### Centralized Single Sign-On (SSO)

Keycloak’s central authentication service supports SSO across web applications,
microservices, APIs, and internal tools. Once users authenticate to Keycloak,
they can use the same session to access multiple applications—significantly
improving user experience and reducing password fatigue.

### User Federation and Identity Brokering

- **User Federation:** Keycloak can connect to existing user directories (like
  LDAP, Active Directory, custom databases), federating user data and
  credentials. This ensures seamless integration with enterprise user stores,
  supports legacy migrations, and enables single identity sources across various
  applications.
- **Identity Brokering:** Keycloak serves as a broker for external identity
  providers (e.g., Google, GitHub, other SAML or OIDC providers). This enables
  social login, cross-organizational authentication, and single sign-on across
  organizations or platforms.

#### LDAP and Active Directory

- **LDAP (Lightweight Directory Access Protocol)**

  - **What it is:** An open, vendor‑neutral protocol used to access and manage
    directory information services over an IP network.
  - **Purpose:** Think of it as the “language” clients use to talk to a
    directory server — retrieving, adding, or modifying entries (like users,
    groups, devices).
  - **Typical Use:** Centralized authentication, contact lists, and application
    directory lookups.
  - **Key trait:** Doesn’t define _how_ the data is stored — just how it’s
    organized (hierarchically) and queried.

- **Active Directory (AD)**

  - **What it is:** Microsoft’s directory service implementation, which stores
    information about objects (users, computers, printers) in a Windows domain.
  - **Purpose:** Provides authentication (login), authorization (access
    control), and policy enforcement in Windows‑based networks.
  - **How it works:** Built on several technologies — notably **LDAP** (for
    directory queries), **Kerberos** (for authentication), and **DNS** (for
    locating services).
  - **Typical Use:** Centralized domain logins, managing user permissions,
    applying Group Policies, and integrating with Microsoft ecosystem services.

### Robust Authentication Flows

- **Customizable and Extensible Flows:** Administrators can define, visualize,
  and modify authentication workflows, supporting steps like password entry,
  OTP, WebAuthn (passkey), social login, or even custom JavaScript/JAR-based
  authenticators.
- **Multi-Factor Authentication (MFA):** Supports TOTP, SMS, WebAuthn for added
  security.
- **Passkey/Passwordless Login:** Supports passwordless authentication and
  passkeys to meet modern usability and security standards.

### Fine-Grained Authorization

- **Role-Based Access Control (RBAC):** Supports both realm roles (global) and
  client roles (application-specific). Roles can be assigned to users, groups,
  or mapped dynamically—enabling strict access policies.
- **Attribute-Based and Policy-Based Access Control:** Advanced authorization
  engine supports policies based on user attributes, environment
  characteristics, custom scripts (ABAC), time, group membership, and
  combined/aggregated strategies.
- **User-Managed Access (UMA):** Implements UMA 2.0 standard, enabling resource
  owners to manage and share their resources with other users via policies.

### Strong Session and Token Management

- **Secure Token Service:** Issues OIDC-compliant ID, access, and refresh
  tokens; supports customizable token lifecycles, revocation, introspection, and
  certificate rotation.
- **Session Management:** Central views, session revocation by user/admin, and
  enforced timeouts.

### Developer and Integration Friendly

- **RESTful and Admin APIs:** Provides programmatic interfaces for managing
  users, clients, roles, realms, and sessions—enabling automation, CI/CD, and
  integration into DevOps workflows.
- **Significant Adapter Support:** Libraries and documentation for integrating
  with Java, JavaScript (SPA), Node.js, Python, .NET, Spring, Quarkus, and more.

### Multi-Tenancy and Realms

- **Realms:** Allows for true multi-tenant setups by separating users, roles,
  clients, and policies into isolated units called realms.

### Major Keycloak Features

| Feature Category         | Description                                                      |
| ------------------------ | ---------------------------------------------------------------- |
| SSO & Auth Flows         | Central login, customizable flows, MFA, passwordless, passkey    |
| Federation & Brokering   | LDAP/AD federation, identity brokering, social login             |
| Authorization            | RBAC, PBAC, ABAC, UMA, client/realm roles, fine-grained policies |
| Session/Token Management | OIDC/SAML tokens, custom expiry, revocation, offline sessions    |
| Theming & Customization  | Theme UI, localization, custom workflows, extensible login flows |
| Extensibility            | Service Provider Interfaces (SPI), custom authenticators/mappers |
| Admin/Dev Experience     | Admin UI/API, RESTful APIs, audit logs, automation               |
| Multi-tenancy            | Realms, clients, user groups, isolated policies                  |
| High Availability        | Clustering, failover, Kubernetes-ready deployments               |
| Compliance               | GDPR, HIPAA, FAPI support, SAML SSO, attribute management        |

---

## Architecture

### High-Level System Architecture

At a macro level, Keycloak is a Java-based application (powered by Quarkus
runtime in recent versions) organized into the following major layers:

- **Core Identity Server:** Implements realm/user/session models,
  authentication, authorization, token issuance, and RESTful APIs.
- **Database Layer:** Supports relational databases (primarily PostgreSQL,
  MySQL, MariaDB, Oracle, etc.) as the durable backend for realms, users, roles,
  and session states.
- **Distributed Cache Layer:** Employs Infinispan to maintain distributed
  session and state caches for high availability and scalability.
- **Frontend Applications:** Admin Console and Account Console (React/PatternFly
  web apps), login/account UIs are served as themes/templates.
- **Service Provider Interfaces (SPIs):** Pluggable extension points allowing
  custom authenticators, protocol mappers, identity providers, storage
  solutions, listeners, and more.

### Core Domain Model

Each **Realm** acts as an isolated unit containing:

- Users
- Groups
- Roles (realm-level and client-level)
- Clients (registered applications/services)
- Identity providers, mappers, and policies
- All relevant authentication and authorization flows

### Storage Layer

Keycloak uses a relational database (e.g., PostgreSQL) for state persistence,
supporting both single-instance and shared-database clustered deployments.

- **User Federation:** Allows for external user stores, providing real-time or
  imported user fetching from LDAP or Active Directory via appropriate federated
  identity providers.

---

## Components

### Realms

A **realm** is a security domain containing its own users, credentials, roles,
clients, policies, and authentication flows. Realms enable true isolation of
users and policies for multi-tenant IAM deployments.

### Clients

A **client** is an application or service (web, mobile, API) that uses Keycloak
for authentication, delegated authorization, or federation. Clients are
registered within a specific realm and configure redirect URIs, scopes, access
types (confidential, public, bearer-only), and protocol support (OIDC, SAML).

### Roles

**Roles** enable access control via RBAC—these may be global (realm roles) or
application-specific (client roles). Roles are assigned directly to users,
groups, or mapped dynamically via external identity providers or protocol
mappers.

### Users and Groups

- **Users:** Individuals, services, or API consumers with credentials and
  identity attributes that can be assigned roles, group memberships, and
  federated identities.
- **Groups:** Organize users with shared roles or policies, enabling mass
  assignment and hierarchical structures.

### Authentication Flows

Keycloak supports graphical, modular, and extensible authentication flows—chains
of authenticators (login form, OTP, browser check, user profile, etc.)—with the
flexibility to add custom steps and conditions through the admin console or SPI
interfaces.

### Identity Providers and User Federation

- **Identity Providers:** OIDC/SAML social providers or corporate identity
  services.
- **User Federation Providers:** LDAP, AD, or custom stores for direct user
  fetching without duplicating the user database.

### Protocol Mappers

Customizes claims/tokens by transforming or adapting data between Keycloak and
applications or external providers. Examples include mapping custom attributes,
group membership, or derived claims.

### Theming and Localization

All user-facing UIs are customizable via themes (built with FreeMarker,
PatternFly, or React for the new consoles)—allowing custom branding, color
schemes, login forms, and localization support for global applications.

### Service Provider Interfaces (SPIs)

Keycloak’s SPI system is the gateway to deep extensibility. Administrators and
plugin developers can add new authenticators, custom storage providers, event
listeners, protocol mappers, identity providers, and custom admin console
sections by implementing and deploying SPI-based JARs to the Keycloak providers
directory.

### Admin and Account Console

- **Admin Console:** A web UI for managing all aspects of Keycloak—realms,
  clients, users, roles, authentication, tokens, themes, and monitoring.
- **Account Console:** End-user dashboard for managing profiles, sessions,
  credentials, linked identities, and settings.

### Monitoring/Auditing

Built-in event listeners provide comprehensive auditing of authentication,
account changes, security events, and administrative actions. Integration with
external log and SIEM systems is also supported.

---

## Authentication Flows

Keycloak’s authentication system is based on flexible and composable flows,
supporting both standard and custom sequences:

### Supported Authentication Flows

- **Standard (Authorization Code) Flow:** The most secure, with browser
  redirection and code exchange (recommended for web apps and SPAs).
- **Implicit Flow:** Legacy; tokens returned in redirect, now discouraged for
  security reasons.
- **Hybrid Flow:** Combines aspects of frontend (implicit) and backend (code
  grant), rarely used.
- **Resource Owner Password Credentials:** For non-interactive flows (not
  recommended for public clients).
- **Client Credentials Flow:** For service-to-service (machine) login; no
  end-user involvement.
- **Device Authorization and CIBA Flows:** For device/browserless and
  backend-initiated authentication, meeting advanced integration needs.

### Multi-Factor and Conditional Authentication

Multiple authenticators can be chained, such as:

- Password + TOTP or SMS code.
- WebAuthn (passwordless or passkey).
- Conditional flows (step-up auth based on risk/context/device/IP/role, etc.).

### Custom Authentication

Using the SPI, custom flows and logic (e.g., company-specific risk engine,
account verification, government eIDs) can be integrated through custom
Authenticators and Actions.

---

## Authorization Services

Keycloak’s advanced authorization system extends RBAC with fine-grained,
policy-based access control:

- **Permissions:** Link resources (APIs, endpoints, objects) with scopes
  (actions, access modes) and enforce policies.
- **Policies:** Define policy logic based on users, roles, groups, clients,
  attributes, environment, JS scripts, regex, aggregated decisions, or custom
  logic using SPI.
- **UMA:** Supports user-controlled resource permission sharing.
- **REST APIs/Java SDK:** Enable dynamic, programmatic management of resources,
  permissions, and entitlement decisions.

---

## Identity Brokering and User Federation

Keycloak offers both federation for legacy/enterprise integration and brokering
for cross-org/social SSO:

- **User Federation:** Integrates with internal directories (LDAP, AD),
  synchronizes or references users and group membership. Supports fetching
  real-time credentials and password validation, with options for cache,
  write-through, and attribute mapping.
- **Identity Brokering:** Allows users to login with a social/corporate IdP.
  Standardizes identity assertion for downstream applications. Supports
  automatic account linking, just-in-time provisioning, and social login
  restriction policies.

---

## Interview Questions

### What is Keycloak and where is it used?

Keycloak is an open-source Identity and Access Management (IAM) system designed
to provide authentication, authorization, user management, SSO, and federation
capabilities to modern applications and microservices. It is used in
microservices security, SSO portals, customer-facing platforms, SaaS,
eGovernment, compliance-driven environments, and anywhere centralized IAM is
required.

### How does Keycloak SSO (Single Sign-On) work?

Keycloak SSO enables users to authenticate once and access multiple applications
and services. Applications are registered as clients in Keycloak, and, upon
login, users are redirected to Keycloak. After successful authentication,
Keycloak issues tokens representing the user’s identity and permissions. These
tokens are trusted and validated by applications, enabling a secure SSO
experience across platforms.

### What are realms and clients in Keycloak?

A Keycloak **realm** is a security domain containing users, clients, roles,
policies, and authentication flows—realms are isolated and support
multi-tenancy. A **client** is an application or service that uses Keycloak for
authentication and authorization. Each client defines its protocol (OIDC, SAML),
redirect URIs, roles, and specific policies.

### How does Keycloak federate users from LDAP or Active Directory?

Keycloak’s user federation feature allows you to connect to external directories
such as LDAP or AD. When a user attempts to authenticate, Keycloak queries the
external directory for credentials and attributes (in real time or via periodic
import). Password policies, group membership, and attribute mappings are
configurable. This allows organizations to maintain a single authoritative
identity source.

### What is the difference between User Federation and Identity Brokering?

User federation provides a direct connection between Keycloak and enterprise
directories for user authentication and lookup (users are fetched from LDAP/AD
as if native). Identity brokering allows users to login via external IdPs
(Google, GitHub, external SAML/OIDC providers) using standard federated
authentication protocols, with Keycloak handling account linking and mapping.

### How does Keycloak enforce authorization?

Keycloak enforces authorization via role mappings (RBAC), attribute-based (ABAC)
or policy-based (PBAC) rules, group memberships, and user-managed permissions
(UMA). Policies may use users, roles, groups, scripts, time, or environmental
context. Permissions can be enforced at API/resource level via scopes, and
evaluated at token issuance or runtime via its Authorization Services module.

### How do you integrate social login into a Keycloak-based solution?

Use built-in Identity Providers:

- In the admin console, add a new identity provider for Google, Facebook, etc.
- Provide their client ID/secret.
- Configure attribute mappings and user linking.
- Selectively enable for realms or applications.
- Users can then sign in using social accounts, and you can control linking,
  creation, and required actions as desired.

### How does Keycloak handle session revocation and logout across multiple applications?

Keycloak maintains a central session for each user. Upon logout (initiated by
the user or admin), Keycloak terminates sessions and notifies all clients
(applications) with registered back-channel logout endpoints. Clients may also
implement front-channel logout or single logout (SLO) based on SAML/OIDC
specifications. Tokens and sessions can also be revoked via admin APIs or CLI.
