---
type: concept
domain: backend
topic: caching
difficulty: hard
status: inbox
tags: [backend]
---

# Caching

## Definition
Storing copies of data closer to the consumer to serve reads faster and offload the source of truth. At the application layer this is the **write/read policy** (cache-aside, write-through, write-behind) plus **invalidation** and **TTL**; at the system layer it's the [[Caching-Strategies|layering]].

## Why it matters
Caching is the highest-leverage latency/scalability lever, and its failure modes (stampede, penetration, avalanche, staleness) cause real incidents. "Cache invalidation is one of the two hard problems" is a cliché because coherence is genuinely hard.

## How it works — the patterns
| Pattern | Read | Write | Consistency | Use |
|---------|------|-------|-------------|-----|
| **Cache-aside** (lazy) | app checks cache → miss → load DB → populate | write DB, invalidate/refresh cache | eventual, simple | default |
| **Write-through** | from cache | write cache + DB synchronously | strong-ish, slower writes | read-heavy, freshness matters |
| **Write-behind** | from cache | write cache, async to DB | risk of loss on crash | write-heavy, tolerant of loss |
| **Read-through** | cache library loads on miss | — | like cache-aside but encapsulated | managed caches |

## Enterprise example — cache-aside with the sharp edges handled
```java
public Product get(long id) {
    Product cached = redis.get(key(id));
    if (cached != null) return cached == NULL_SENTINEL ? null : cached;   // negative caching
    // single-flight: prevent stampede when a hot key expires
    return loader.loadOnce(key(id), () -> {
        Product p = repo.findById(id).orElse(null);
        redis.set(key(id), p == null ? NULL_SENTINEL : p, ttlWithJitter(10, MINUTES)); // avalanche guard
        return p;
    });
}
```

## The failure modes (senior differentiator)
- **Stampede/dogpile**: hot key expires → many concurrent misses hit the DB. Fix: per-key single-flight lock, probabilistic early expiration, stale-while-revalidate.
- **Penetration**: lookups for non-existent keys always miss (or a scan). Fix: **negative caching** (short TTL), Bloom filter.
- **Avalanche**: many keys expire simultaneously (same TTL). Fix: **jitter** TTLs, staggered warmup.
- **Stale writes race**: read-then-cache can repopulate a value that a concurrent write just changed. Fix: invalidate-on-write + short TTL, or versioned keys.

## Invalidation strategies
TTL-only (simple, bounded staleness), invalidate-on-write (delete key; next read repopulates), or update-on-write. Choose the acceptable staleness *per dataset* — a price list vs a product description have different tolerances.

## Trade-offs
- Latency/scale gains vs staleness + complexity; local cache = fastest but per-node incoherent; distributed ([[Redis]]) = coherent + a hop + a dependency.

## Common mistakes (senior-level)
- No TTL and no invalidation → permanent staleness.
- Uniform TTLs → avalanche.
- No stampede protection on hot keys.
- Not caching negatives → penetration by repeated missing-key lookups.
- Treating cache as source of truth / no graceful degradation when it's down (a cache outage shouldn't take down the app — fall through to DB, maybe shed load).

## Interview questions (staff+)
- Cache-aside vs write-through vs write-behind — consistency and use.
- Explain stampede, penetration, avalanche and mitigate each.
- How do you invalidate safely under concurrent reads/writes?
- What happens to your system when the cache goes down, by design?

## Related concepts
- [[Caching-Strategies]]
- [[Redis]]
- [[HTTP]]
