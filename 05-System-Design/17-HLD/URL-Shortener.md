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
Map long URLs to short codes and redirect codes back to the original — a read-dominated system (redirects ≫ creates).

## 2. Requirements
### Functional
- Create a short code for a long URL; optional custom alias and expiry.
- Redirect `GET /{code}` to the original.
- Optional click analytics.
### Non-functional
- Redirect p99 < ~20 ms; very high availability; durable mappings; scale reads massively.

### Capacity estimation (back-of-envelope)
```
Writes: 100M new URLs/month ≈ 100M / (30×86400) ≈ 40 writes/s (peak ~5× = 200/s)
Read:write ≈ 100:1  -> reads ≈ 4,000/s (peak ~20,000/s)
Storage/row ≈ 500 B (code, long_url, meta) -> 100M/mo × 500B ≈ 50 GB/mo ≈ 600 GB/yr
5-year mappings ≈ 6B rows -> key space must exceed this comfortably
Cache: 20% of URLs drive 80% of reads -> cache ~top few GB in Redis for high hit ratio
```

## 3. API design
```text
POST /api/v1/urls {url, customAlias?, ttl?} -> 201 {code, shortUrl}
GET  /{code} -> 301 (permanent, cacheable) or 302 (if analytics needed on each hit)
```
301 is cacheable by browsers/CDN (fewer backend hits, but you lose per-hit analytics); 302 forces a hit (analytics, but more load). Pick per requirement.

## 4. Data model
`urls(code PK, long_url, owner_id, created_at, expires_at)` — KV-shaped, keyed by `code`.

## 5. High-level architecture
Client → CDN/L7 [[Load-Balancing|LB]] → stateless app → [[Redis]] (code→url) → DB (+ read replicas). Analytics events go async to a [[Message-Queues|queue]].

## 6. Component responsibilities
- **Encoding service**: generate a unique short code.
- **Redirect service**: cache-first lookup, then DB; emit analytics async.

## 7. Database choice
KV store or relational keyed by `code`; [[Replication|read replicas]] for read scale, [[Sharding]] by `code` only if one primary can't hold the write/storage.

## 8. Cache strategy
Aggressive cache-aside ([[Caching-Strategies]]); mappings are immutable so staleness is a non-issue → near-100% hit ratio for hot codes; jitter TTLs; cache negatives to resist scanning ([[API-Security]]).

## 9. Messaging strategy
Redirects must stay fast, so click analytics are fire-and-forget to a queue → aggregated offline.

## 10. Scaling strategy
Stateless app autoscale + cache + read replicas handles the 100:1 read skew. Code generation is the only stateful concern.

## Code generation approaches (the core design question)
| Approach | Pros | Cons |
|----------|------|------|
| **Counter + Base62** | short, no collisions, dense | sequential = guessable/enumerable; needs a distributed counter |
| **Random Base62** | unguessable | must check collision (extra read) as space fills |
| **Hash(url) truncated** | dedupes same URL | collisions need handling; not customizable |
- Base62 (`[0-9A-Za-z]`) gives 62^7 ≈ 3.5 trillion codes in 7 chars — ample for 6B rows.
- Distributed counter without a hotspot: hand out **ranges** (e.g. ZooKeeper/DB sequence blocks) per node, or use a Snowflake-style ID then Base62-encode.

## 11. Failure scenarios
Cache miss → DB; replica lag is harmless (mappings immutable); DB down → serve from cache, degrade creates.

## 12. Consistency decisions
Mappings are immutable → strong consistency is trivial; analytics are eventually consistent.

## 13. Security considerations
Validate/normalize URLs, block open-redirect abuse and malicious targets, [[Rate-Limiting]] on create, cache negative lookups.

## 14. Observability
Redirect latency, cache hit ratio, code-gen collisions, 4xx/5xx ([[Observability]]).

## 15. Bottlenecks
Code-gen coordination (solved by ranges/Snowflake) and hot-key redirects (solved by cache/CDN).

## 16. Trade-offs
301 vs 302 (caching vs analytics); counter (dense, guessable) vs random (unguessable, collision checks).

## 17. Interview discussion points
Code-generation scheme + distributed counter, cache sizing from the 80/20 read skew, custom aliases, expiry/cleanup (TTL vs batch purge).

## Related concepts
- [[Caching-Strategies]]
- [[Redis]]
- [[Load-Balancing]]
- [[Scalability]]
