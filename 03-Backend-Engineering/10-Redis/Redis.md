---
type: concept
domain: backend
topic: redis
difficulty: hard
status: inbox
tags: [backend]
---

# Redis

## Definition
An in-memory data-structure store used as a cache, broker, and coordination primitive. Command execution is effectively **single-threaded** (one command at a time → per-command atomicity), with rich types (strings, hashes, lists, sets, sorted sets, streams, bitmaps, HyperLogLog) and optional persistence.

## Why it matters
Redis is the default for [[Caching]], [[Distributed-Locks]], [[Rate-Limiting-Design|rate limiting]], leaderboards, sessions, and [[Idempotency]] keys. Architect depth means understanding its threading model, persistence/durability trade-offs, eviction, and its failure behavior in your system.

## How it works — the mechanism
- **Single-threaded command loop**: commands are serialized → each is atomic; no locks needed for simple ops. (Redis 6+ uses threaded I/O for network, but command *execution* is still serialized.) → **avoid O(n) commands** (`KEYS *`, big `SMEMBERS`) that block the loop and stall everyone.
- **Atomic multi-step**: `MULTI/EXEC` (queued, no rollback) or **Lua scripts** (run atomically on the loop — the correct way to do check-and-set for locks/limits).
- **Persistence**:
  - **RDB**: periodic point-in-time snapshot — compact, fast restart, but loses writes since the last snapshot.
  - **AOF**: append-only log of writes — better durability (fsync every sec / always), larger, slower restart.
  - Neither makes Redis a system-of-record; treat it as a cache/coordination layer.
- **Eviction** (when `maxmemory` hit): `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `noeviction`, etc. — pick per use (a pure cache wants `allkeys-lru`).
- **HA/scale**: Sentinel (failover) or **Cluster** (hash-slot sharding across nodes, 16384 slots).

## Enterprise example — atomic building blocks
```text
SET session:{id} <blob> EX 1800                 # session, 30-min TTL
INCR rl:{user}:{window}  (+ EXPIRE)             # rate-limit counter (atomic)
SET lock:{key} <token> NX PX 5000               # distributed lock w/ TTL (see Distributed-Locks)
ZADD leaderboard <score> <member>               # sorted set for ranking
```

## Trade-offs
- Blazing fast (in-memory) but memory-bound and not durable like an RDBMS; simple data model.
- Single-threaded → one slow command hurts everyone (latency spikes from `KEYS`, big ranges, or Lua that loops).
- Cluster adds scale but multi-key ops must share a hash slot (`{tag}` hash tags).

## Common mistakes (senior-level)
- Treating Redis as the source of truth without durability planning.
- Running O(n) commands (`KEYS *`, huge `LRANGE`) in production → loop stalls.
- Naive distributed locks without token/TTL/fencing ([[Distributed-Locks]]).
- No plan for a Redis outage — cache misses should **degrade gracefully to the DB**, not cascade; a lock/limit store outage needs a fail-open/closed decision.
- Ignoring eviction policy → unexpected key loss or OOM.

## Interview questions (staff+)
- Why is Redis single-threaded, and what does that imply for command choice?
- RDB vs AOF — durability vs performance trade-offs.
- How do you do an atomic check-and-set (Lua) and why not GET+SET?
- What happens to your system when Redis goes down (cache vs lock vs limit)?
- How does Redis Cluster shard, and what constrains multi-key ops?

## Related concepts
- [[Caching]]
- [[Distributed-Locks]]
- [[Rate-Limiting-Design]]
- [[Idempotency]]
