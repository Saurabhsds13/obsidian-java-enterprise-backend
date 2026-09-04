---
type: concept
domain: system-design
topic: caching
difficulty: hard
status: inbox
tags: [system-design]
---

# Caching Strategies

## Definition
System-level patterns for *where* to place caches (client, CDN, application, database) and *how* to keep them coherent (write policy, invalidation, TTL). At the application layer it maps to the [[Caching|read/write patterns]]; at the system layer it's about layering and failure behavior.

## Why it matters
Caching is the single highest-leverage latency/scalability lever, and "there are only two hard things: cache invalidation and naming things" is a cliché because it's true. Senior interviews probe the failure modes — stampede, penetration, avalanche — not just "add a cache."

## How it works — the layers
```
Browser cache  ->  CDN/edge  ->  App cache (local / Redis)  ->  DB buffer pool
   cheapest, closest to user  ------------------------------>  source of truth
```
Write/coherence policies (see [[Caching]]): **cache-aside** (default), **write-through** (consistent, slower writes), **write-behind** (fast, risk of loss), plus **TTL** expiry and **explicit invalidation** on write.

## The failure modes (the senior differentiator)
| Problem | What happens | Mitigation |
|---------|--------------|------------|
| **Stampede / dogpile** | hot key expires → thousands of concurrent misses hammer the DB | per-key lock / single-flight; probabilistic early refresh; stale-while-revalidate |
| **Penetration** | requests for keys that don't exist bypass cache every time (or a malicious scan) | cache negative results (short TTL); Bloom filter of valid keys |
| **Avalanche** | many keys expire at once (same TTL) → mass DB load | jitter TTLs; staggered warmup |
| **Hot key** | one key's traffic exceeds a single node | local cache in front of Redis; key replication/splitting |

## Enterprise example — cache-aside with stampede protection
```java
public Product get(long id) {
    Product p = redis.get(key(id));
    if (p != null) return p;                 // hit
    // single-flight: only one loader per key (Redis SETNX lock or in-proc)
    return loader.load(key(id), () -> {
        Product fresh = repo.findById(id).orElse(Product.MISSING);   // cache negatives too
        redis.set(key(id), fresh, ttlWithJitter(Duration.ofMinutes(10)));
        return fresh;
    });
}
```

## Consistency: how do you invalidate?
- **TTL-only**: simplest; bounded staleness; no write coordination.
- **Write-through / update-on-write**: fresher but couples writes to the cache.
- **Invalidate-on-write** (delete the key): common; next read repopulates. Beware the read-then-write race (a concurrent read can repopulate a stale value) — mitigate with short TTL or versioned keys.

## Trade-offs
- Latency/scale vs staleness and added complexity; each layer is another invalidation surface and another failure mode.
- Local cache = fastest but per-node inconsistency; distributed cache ([[Redis]]) = coherent but a network hop and a dependency.

## Common mistakes (senior-level)
- No TTL and no invalidation → permanent staleness.
- Identical TTLs → avalanche.
- No stampede protection on hot keys.
- Not caching negatives → penetration by repeated missing-key lookups.
- Treating the cache as the source of truth / no graceful degradation when it's down.

## Interview questions (staff+)
- Walk the cache layers and pick where to cache for a given read.
- Explain stampede, penetration, avalanche — and mitigate each.
- Cache-aside vs write-through vs write-behind, with consistency implications.
- How do you invalidate safely under concurrent reads/writes?

## Related concepts
- [[Caching]]
- [[Redis]]
- [[Load-Balancing]]
- [[Scalability]]
