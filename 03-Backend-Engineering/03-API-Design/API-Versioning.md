---
type: concept
domain: backend
topic: api-design
difficulty: medium
status: inbox
tags: [backend]
---

# API Versioning

## Definition
Strategies for evolving an API while keeping existing clients working: **URI** versioning (`/v2/...`), **header/media-type** versioning (`Accept: application/vnd.acme.v2+json`), or **query-param** (`?version=2`). The deeper skill is knowing what counts as breaking and avoiding version churn entirely.

## Why it matters
Clients you don't control depend on your contract. A breaking change without a version is a production incident for every integrator. But versioning has a maintenance cost, so an architect versions *deliberately*, not reflexively.

## How it works — the strategies
| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| **URI** | `/api/v2/orders` | visible, cache-friendly, trivial to route/test | "un-RESTful" (URL identifies representation, not resource); duplicates routes |
| **Header / media type** | `Accept: ...v2+json` | clean URLs, content negotiation | harder to test/debug/cache; less discoverable |
| **Query param** | `?version=2` | simple | easy to omit; caching quirks |

URI versioning is the pragmatic enterprise default.

### What is breaking vs non-breaking (the core knowledge)
- **Breaking**: removing/renaming a field, changing a type, tightening validation, changing default behavior, removing an endpoint, changing error semantics.
- **Non-breaking (additive)**: new optional request fields, new response fields, new endpoints, new optional query params.
- **Tolerant reader** principle: clients must ignore unknown fields → lets you add fields without a version bump.

### Deprecation lifecycle
Announce → run old + new in parallel → emit `Deprecation`/`Sunset` headers + usage metrics → migrate clients → remove after a defined window. Never yank a version with active traffic.

## Enterprise example
```java
@RestController
@RequestMapping("/api/v2/orders")   // URI versioning; v1 controller kept until sunset
class OrderControllerV2 { ... }
// Deprecation signalling on the old version:
// Deprecation: true
// Sunset: Sat, 01 Aug 2026 00:00:00 GMT
```

## Trade-offs
- More versions = more maintenance surface (code, tests, docs). Version only on **true breaking changes**; prefer additive evolution.
- Header versioning is elegant but operationally harder (caching, debugging, gateway routing).

## Common mistakes (senior-level)
- Bumping the version for additive changes (unnecessary churn).
- Making breaking changes silently within a version.
- Removing an old version with no deprecation window / no usage telemetry.
- Not designing clients as tolerant readers, forcing versions for trivial additions.

## Interview questions (staff+)
- URI vs header vs query versioning — trade-offs and your default.
- What exactly is a breaking change? Give examples of additive changes.
- How does the tolerant-reader principle reduce versioning?
- Describe a safe deprecation lifecycle.

## Related concepts
- [[REST]]
- [[HTTP]]
