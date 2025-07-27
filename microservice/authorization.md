---
title: Authorization
---

## Authorization

### Access Control List (ACL)

An ACL is a per-resource list, each resource (like a file, API endpoint or graph
node) stores entries `<principal, permissions>` that explicitly grant or deny
actions (e.g., `alice:read,write`, `bob:read only`)

Good for file-storage systems or service-specific permissions at object level
(e.g. giving specific users access to a data row or bucket).

### Role-Based Access Control (RBAC)

Users are assigned _roles_ (e.g. `admin`, `editor`), and roles map to sets of
permissions. Access is decided based on the user's assigned roles.

Use within admin dashboards, corporate web apps, or when roles map cleanly to
job functions.

### Policy-Based Access Control (PBAC)

PBAC centralizes authorization in _human-readable policies_, evaluated at
runtime by a policy engine. Policies consider roles, plus attributes of
subject/resource/action/context (e.g. time, location).

Ideal for modern distributed systems where policies need updating across many
microservices, especially when dynamic, attribute-aware decisions matter (e.g.
time restrictions, multi-tenancy, geo-based rules).

```
allow {
  input.user.role == "manager"
  input.resource.owner == input.user.id
  input.context.time < resources[input.resource.id].expiry
}
```

## Interview Questions

### What is an ACL (Access Control List) and when might you use it?

An ACL is a per‑resource list of `<principal, permissions>` entries (e.g.
`alice:read`, `bob:write`). It grants or denies access at the individual
resource level. ACLs are straightforward and fine‑grained, but poorly scalable
as resources grow and management becomes complex.

**Use case**: Ideal for small systems or object‑level controls like individual
document sharing or simple file storage (e.g. S3 bucket ACLs).

### How does RBAC work and what are its benefits and limitations?

RBAC assigns users to roles (e.g. admin, editor), and roles carry a set of
permissions. Access is determined by the roles a user has.

**Benefits**: Scalable for organizational structures, simplifies management,
supports segregation of duties, easier auditing.

**Limitations**: Can lead to role explosion when trying to model fine‑grained or
contextual access (many roles needed per scenario).

### What is PBAC, and how is it different from RBAC or ACL?

Policy-Based Access Control (PBAC) defines access rules in high-level policies that may refer to roles, attributes, and environmental context. An engine evaluates these policies at runtime.

- **Unlike ACL**, PBAC does not require per‑resource entries.

- **Unlike RBAC**, it can consider context like time, IP, resource metadata, or
  relationship attributes. PBAC enables dynamic, flexible, auditable access logic
  in one expressible language or format .

### Can you give a web‑app scenario modeling ACL vs RBAC vs PBAC?

- ACL: In a file‑sharing CRUD API, each file record stores ACL entries granting
  users read/write.

- RBAC: A CMS web admin assigns users roles like `editor` or `publisher`. The
  backend middleware grants permissions based on user role claims.

- PBAC: A microservices authorisation engine reads policies like `allow if
user.role == "manager" and resource.owner == user.id` or `allow if
user.department == resource.department and within working hours`, and evaluates
  using JSON‑based attribute inputs and a policy decision point (PDP).
