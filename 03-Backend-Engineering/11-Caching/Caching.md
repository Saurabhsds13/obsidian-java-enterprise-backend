---
type: concept
domain: backend
topic: caching
difficulty: medium
status: inbox
tags: [backend]
---

# Caching

## Definition
Storing copies of data closer to the consumer to serve reads faster and reduce load on the source of truth.

## Why it matters
Caching is the highest-leverage latency and scalability tool — and cache invalidation is famously one of the hard problems.

## How it works
Common patterns:
- **Cache-aside** (lazy): app reads cache; on miss, loads from DB and populates cache. Most common.
- **Write-through**: writes go to cache and DB synchronously.
- **Write-behind**: writes to cache, async to DB (fast, risk of loss).

```java
public Product get(long id) {
    Product cached = redis.get(key(id));
    if (cached != null) return cached;
    Product p = repo.findById(id).orElseThrow();
    redis.set(key(id), p, Duration.ofMinutes(10));   // TTL guards staleness
    return p;
}
```

## Production usage
Use TTLs to bound staleness; invalidate on writes; guard against **stampede** (locking / request coalescing) and **penetration** (cache misses for nonexistent keys). Back with [[Redis]].

## Trade-offs
- Speed vs consistency: caches serve stale data by design; choose acceptable staleness.

## Common mistakes
- No TTL and no invalidation (permanent staleness).
- Cache stampede on hot-key expiry.

## Interview questions
- Cache-aside vs write-through?
- How do you handle cache stampede and invalidation?

## Related concepts
- [[Redis]]
- [[Caching-Strategies]]
- [[HTTP]]
