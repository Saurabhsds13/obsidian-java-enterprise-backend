---
type: system-design
level: HLD
domain: system-design
difficulty: medium
status: inbox
tags: [system-design]
---

# URL Shortener

## 1. Problem
Turn long URLs into short codes that redirect back to the original.

## 2. Requirements
### Functional
- Create a short code for a long URL; optional custom alias.
- Redirect a short code to the original URL.
- Optional analytics (click counts).
### Non-functional
- Very read-heavy (redirects ≫ creations); low-latency redirects; high availability.
### Scale assumptions
- e.g. 100M new URLs/month, ~10:1000 write:read ratio.

## 3. API design
```text
POST /api/v1/urls { "url": "https://..." } -> { "code": "abc123" }
GET  /{code} -> 301/302 redirect
```

## 4. Data model
`urls(code PK, long_url, created_at, owner_id, expires_at)`

## 5. High-level architecture
Client → LB ([[Load-Balancing]]) → stateless app → cache ([[Redis]]) → DB. Redirects served mostly from cache.

## 6. Component responsibilities
- Encoding service: generates unique codes.
- Redirect service: cache-first lookup then DB.

## 7. Database choice
Key–value or relational keyed by `code`; read replicas ([[Replication]]) for scale.

## 8. Cache strategy
Cache `code → long_url` aggressively ([[Caching-Strategies]]); hot codes rarely change.

## 9. Messaging strategy
Async analytics via a queue ([[Message-Queues]]) so redirects stay fast.

## 10. Scaling strategy
Stateless app + cache + read replicas; shard by code if needed ([[Sharding]]).

## 11. Failure scenarios
Cache miss → DB; DB replica lag tolerable (URLs immutable).

## 12. Consistency decisions
URLs are immutable once created → strong consistency easy; analytics eventually consistent.

## 13. Security considerations
Validate/normalize URLs, block open-redirect abuse and malicious targets, [[Rate-Limiting]] on create.

## 14. Observability
Redirect latency, cache hit ratio, 4xx/5xx rates ([[Observability]]).

## 15. Bottlenecks
Code generation uniqueness; hot-key redirects (solved by cache).

## 16. Trade-offs
Counter/base62 (sequential, guessable) vs hash (collision handling) vs random + uniqueness check.

## 17. Interview discussion points
Code generation approaches, cache sizing, custom aliases, expiry/cleanup.

## Related concepts
- [[Caching-Strategies]]
- [[Redis]]
- [[Load-Balancing]]
