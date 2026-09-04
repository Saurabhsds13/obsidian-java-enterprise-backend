---
type: concept
domain: backend
topic: authorization
difficulty: hard
status: inbox
tags: [backend]
---

# Authorization

## Definition
Deciding **what** an authenticated principal may do — enforcing permissions on resources and actions. It runs *after* [[Authentication]] and is where "broken access control" (OWASP #1) usually happens.

## Why it matters
Authentication without correct authorization = privilege escalation and data leaks. The most common real vulnerability is **IDOR** (Insecure Direct Object Reference / broken object-level authorization): checking that a user is logged in but not that they own the object they're accessing.

## How it works — the models
| Model | Decision basis | Pros | Cons |
|-------|---------------|------|------|
| **RBAC** | roles → permissions; users have roles | simple, auditable | coarse; role explosion for fine rules |
| **ABAC** | attributes (owner, dept, time, resource tags) | expressive, contextual | complex to reason about/audit |
| **ReBAC** | relationships (owner-of, member-of) graph | models sharing (Google Docs-style) | needs a graph/policy engine (e.g. Zanzibar) |

### Enforce at two levels
- **Coarse (URL/method)**: `hasRole('ADMIN')` on an endpoint.
- **Fine (object-level)**: verify the principal owns/may access *this specific* resource — the check that prevents IDOR.

```java
@PreAuthorize("hasRole('ADMIN') or #ownerId == authentication.name")   // coarse + ownership
public Account get(String ownerId) { ... }

// Object-level check in the service — NEVER trust a client-supplied id alone:
Order o = orders.findById(id).orElseThrow();
if (!o.ownerId().equals(currentUser.id())) throw new AccessDeniedException();
```

## Principles
- **Least privilege**: grant the minimum needed; deny by default.
- **Server-side enforcement**: never rely on the UI hiding a button — the API must enforce.
- **Fail closed**: an error in the authz check denies, not allows.

## Trade-offs
- RBAC is simple but coarse; ABAC/ReBAC are expressive but harder to audit and reason about. Start with RBAC + ownership checks; add ABAC/ReBAC only when the domain needs it.

## Common mistakes (senior-level)
- **IDOR**: `GET /orders/{id}` returns any order because ownership isn't checked (only that the user is logged in).
- Authorization only in the UI / gateway, not in the service.
- Trusting client-supplied identifiers (user id in the body) instead of the authenticated principal.
- Fail-open on authz errors.
- Role explosion — encoding fine-grained rules as ever more roles instead of attributes.

## Interview questions (staff+)
- RBAC vs ABAC vs ReBAC — when each?
- What is IDOR / broken object-level authorization and how do you prevent it?
- Why enforce authorization in the service, not just the gateway/UI?
- How do least-privilege and fail-closed apply to an authz check?

## Related concepts
- [[Authentication]]
- [[API-Security]]
- [[Spring-Security]]
- [[JWT]]
