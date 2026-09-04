---
type: concept
domain: backend
topic: rest
difficulty: medium
status: inbox
tags: [backend]
---

# REST

## Definition
An architectural style for networked APIs: **resources** (nouns) identified by URLs, manipulated with uniform [[HTTP]] methods, exchanged as representations (usually JSON), with **stateless** interactions. "RESTful" is a spectrum (Richardson Maturity Model), not a binary.

## Why it matters
REST is the default API style. A consistent, predictable, evolvable contract reduces client friction and support cost — and interviewers probe whether you can design one that stays backward-compatible under change.

## How it works — the mechanism

### Richardson Maturity Model (levels of "RESTful")
- **L0**: one endpoint, RPC-over-HTTP (POST everything).
- **L1**: resources (`/orders`, `/orders/42`).
- **L2**: HTTP verbs + status codes correctly (most real APIs live here).
- **L3**: HATEOAS (responses embed links to next actions) — rare in practice; often not worth the cost.

### Core constraints
- **Resource modeling**: nouns not verbs (`POST /orders`, not `POST /createOrder`); nest for relationships (`/orders/42/items`).
- **Uniform interface**: verbs mean what HTTP says ([[HTTP]]); status codes are honest.
- **Stateless**: each request self-contains auth + context (token, params) → any node serves it ([[Load-Balancing]]).
- **Collection ergonomics**: [[Pagination]], filtering, sorting, sparse fieldsets.

### Evolvability (the senior focus)
- **Non-breaking**: add optional fields, add endpoints, add enum values (if clients tolerate unknowns).
- **Breaking**: remove/rename fields, change types, tighten validation, change semantics → needs [[API-Versioning]].
- Prefer additive evolution + tolerant readers (ignore unknown fields) to avoid version churn.

## Enterprise example — a clean resource contract
```text
GET    /api/v1/orders?status=SETTLED&page=2&size=50&sort=createdAt,desc
POST   /api/v1/orders            -> 201 Created + Location: /api/v1/orders/99  (Idempotency-Key)
GET    /api/v1/orders/99
PATCH  /api/v1/orders/99         -> partial update
DELETE /api/v1/orders/99         -> 204
Errors: RFC 7807 problem+json { type, title, status, detail, instance }
```

## Trade-offs
| Style | Strength | Weakness |
|-------|----------|----------|
| REST | simple, cacheable, ubiquitous | over/under-fetching, chatty for graphs |
| GraphQL | flexible client-shaped queries | caching, complexity, N+1 on resolvers |
| gRPC | fast binary, streaming, contracts | not browser-native, less human-debuggable |

## Common mistakes (senior-level)
- Verbs in URLs (`/getOrders`, `/updateOrder`).
- Breaking changes without a version bump.
- Inconsistent error shapes (adopt RFC 7807).
- Ignoring idempotency for unsafe retries ([[Idempotency]]).
- Chasing full HATEOAS (L3) when L2 + good docs suffices.

## Interview questions (staff+)
- What makes an API RESTful (Richardson levels)?
- How do you evolve an API without breaking clients (additive + tolerant readers + versioning)?
- REST vs GraphQL vs gRPC — pick one for a scenario.
- Why statelessness, and where does session state go?

## Related concepts
- [[HTTP]]
- [[Idempotency]]
- [[Pagination]]
- [[API-Versioning]]
