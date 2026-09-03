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
Strategies for evolving an API while keeping existing clients working: URI versioning, header/media-type versioning, or query-param versioning.

## Why it matters
Clients depend on your contract. Breaking changes without versioning break integrations in production.

## How it works
- **URI**: `/api/v1/orders` → `/api/v2/orders`. Most visible and cache-friendly.
- **Header / media type**: `Accept: application/vnd.myapp.v2+json`. Cleaner URLs, harder to test/debug.
- **Query param**: `?version=2`. Simple but easy to omit.

## What counts as breaking
Breaking: removing/renaming fields, changing types, tightening validation. Non-breaking: adding optional fields/endpoints. Prefer additive evolution to avoid new versions.

## Production usage
URI versioning is the common enterprise default. Support the previous version during a deprecation window; announce timelines; log usage of deprecated versions.

## Trade-offs
- More versions = more maintenance. Version only on true breaking changes.

## Common mistakes
- Versioning for additive changes.
- Removing an old version with no deprecation period.

## Interview questions
- How do you evolve an API without breaking clients?
- URI vs header versioning trade-offs?

## Related concepts
- [[REST]]
- [[HTTP]]
