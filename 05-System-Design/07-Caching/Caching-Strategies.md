---
type: concept
domain: system-design
topic: caching
difficulty: medium
status: inbox
tags: [system-design]
---

# Caching Strategies

## Definition
System-level patterns for placing and maintaining caches (client, CDN, application, database) to reduce latency and load.

## Why it matters
Caching is the highest-leverage scalability tool in system design, and choosing the right layer and invalidation approach is a frequent interview focus.

## How it works
Layers:
- **Client / browser**: cache responses via [[HTTP]] headers.
- **CDN / edge**: cache static and cacheable dynamic content near users.
- **Application cache**: in-memory or [[Redis]] ([[Caching|cache-aside]]).
- **Database cache**: buffer pool, query cache.

Write/consistency approaches mirror application [[Caching]]: cache-aside, write-through, write-behind, plus TTL-based expiry and explicit invalidation.

## Production usage
Cache the hottest, most-read, least-changing data. Guard against stampede (locking/coalescing) and hot keys. Decide the acceptable staleness per dataset.

## Trade-offs
- Latency/scale gains vs staleness and added complexity; more layers = more invalidation surfaces.

## Common mistakes
- Caching everything, including cheap or highly-volatile data.
- No invalidation strategy.

## Interview questions
- Where would you place caches in this design?
- How do you keep caches consistent?

## Related concepts
- [[Caching]]
- [[Redis]]
- [[Load-Balancing]]
- [[Scalability]]
