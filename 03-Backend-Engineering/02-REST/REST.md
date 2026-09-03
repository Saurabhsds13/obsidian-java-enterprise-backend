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
An architectural style for networked APIs using resources (nouns) identified by URLs, manipulated with standard [[HTTP]] methods, and represented (usually) as JSON.

## Why it matters
REST is the dominant API style. A consistent, predictable contract reduces client friction and support cost.

## How it works
- **Resources & URLs**: `/orders/42`, `/orders/42/items`.
- **Methods map to actions**: GET read, POST create, PUT/PATCH update, DELETE remove.
- **Stateless**: each request carries all it needs (auth token, params).
- Support [[Pagination]], filtering, sorting, [[API-Versioning]], and consistent error bodies.

```text
GET  /api/v1/orders?status=SETTLED&page=2&size=50&sort=createdAt,desc
POST /api/v1/orders            -> 201 Created + Location
```

## Production usage
Version the contract, use [[Idempotency]] keys for unsafe retries, return RFC 7807 problem details on errors, and keep backward compatibility (add fields, don't remove/rename).

## Trade-offs
- Simple and cacheable; can be chatty. GraphQL/gRPC fit other needs (flexible queries, low-latency RPC).

## Common mistakes
- Verbs in URLs (`/getOrders`).
- Breaking changes without a version bump.

## Interview questions
- What makes an API RESTful?
- How do you evolve an API without breaking clients?

## Related concepts
- [[HTTP]]
- [[Idempotency]]
- [[Pagination]]
- [[API-Versioning]]
