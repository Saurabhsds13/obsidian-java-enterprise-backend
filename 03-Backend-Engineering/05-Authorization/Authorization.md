---
type: concept
domain: backend
topic: authorization
difficulty: medium
status: inbox
tags: [backend]
---

# Authorization

## Definition
Deciding **what** an authenticated principal is allowed to do — enforcing permissions on resources and actions.

## Why it matters
Authentication without correct authorization leads to privilege escalation and data leaks (broken access control is a top web risk).

## How it works
- **RBAC** (role-based): permissions attached to roles; users have roles.
- **ABAC** (attribute-based): decisions from attributes (owner, department, time).
- Enforce at the edge (URL rules) and in the service layer (method security, ownership checks).

```java
@PreAuthorize("hasRole('ADMIN') or #ownerId == authentication.name")
public Account get(String ownerId) { ... }
```

## Production usage
Apply least privilege. Always re-check ownership server-side (never trust client-supplied IDs). Combine coarse URL rules with fine-grained domain checks.

## Trade-offs
- RBAC is simple but coarse; ABAC is expressive but harder to reason about and audit.

## Common mistakes
- Checking only at the UI/URL, not on the object (IDOR / broken object-level authorization).

## Interview questions
- RBAC vs ABAC?
- How do you prevent IDOR?

## Related concepts
- [[Authentication]]
- [[Spring-Security]]
- [[API-Security]]
