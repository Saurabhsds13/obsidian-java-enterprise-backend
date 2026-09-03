---
type: concept
domain: backend
topic: redis
difficulty: medium
status: inbox
tags: [backend]
---

# Redis

## Definition
An in-memory data store used as a cache, message broker, and data structure server (strings, hashes, lists, sets, sorted sets, streams).

## Why it matters
Redis is the default choice for caching, distributed locks, rate limiting, and session storage in backend systems.

## How it works
- Single-threaded command execution → atomic per-command operations.
- Persistence via RDB snapshots and/or AOF log.
- TTLs and eviction policies (`allkeys-lru`, etc.) bound memory.
- Building blocks: `SETNX`/`SET NX` (locks, [[Idempotency]]), sorted sets (leaderboards, rate limits), `INCR` (counters).

```text
SET session:abc "..." EX 1800          # session with 30-min TTL
INCR rate:user:42                      # rate-limit counter
SET lock:order:99 <token> NX PX 5000   # distributed lock with expiry
```

## Production usage
Cache-aside layer ([[Caching]]), distributed locks ([[Distributed-Locks]]), [[Rate-Limiting]], session storage. Plan for failure: cache misses should degrade gracefully to the DB, not cascade.

## Trade-offs
- Blazing fast but memory-bound and (by default) not durable like an RDBMS. Data structures are simple.

## Common mistakes
- Treating Redis as the source of truth without durability planning.
- Naive locks without expiry/fencing tokens.

## Interview questions
- How would you build a distributed lock with Redis? What can go wrong?
- What happens to your system when Redis goes down?

## Related concepts
- [[Caching]]
- [[Distributed-Locks]]
- [[Rate-Limiting]]
- [[Idempotency]]
