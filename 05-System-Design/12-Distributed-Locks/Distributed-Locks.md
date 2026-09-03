---
type: concept
domain: system-design
topic: distributed-locks
difficulty: hard
status: inbox
tags: [system-design]
---

# Distributed Locks

## Definition
A mechanism to ensure only one process across many machines performs a critical section at a time.

## Why it matters
Single-JVM locks ([[synchronized]]) don't work across instances. Distributed locks coordinate exclusive work (leader-only jobs, preventing double processing).

## How it works
- **Redis**: `SET key token NX PX <ttl>` acquires; release only if the token matches (compare-and-delete via Lua) to avoid deleting someone else's lock. TTL prevents deadlock if the holder crashes.
- **ZooKeeper/etcd**: ephemeral nodes + consensus give stronger guarantees.
- **Fencing tokens**: a monotonically increasing token guards against a paused holder acting after its lock expired.

```text
SET lock:job42 <uuid> NX PX 10000   # acquire with 10s TTL
# ... do work (shorter than TTL) ...
# release iff value == <uuid>
```

## Production usage
Prefer alternatives first: idempotency ([[Idempotency]]), unique DB constraints, or partitioning work by key ([[Kafka]] partitions). Use a lock only when truly needed; keep critical sections short.

## Trade-offs
- Simplicity (Redis) vs correctness under failures. TTL too short → double execution; too long → stalls.

## Common mistakes
- Deleting a lock you no longer own.
- Assuming a lock guarantees safety without fencing under GC pauses/clock skew.

## Interview questions
- How would you build a distributed lock and what can go wrong?
- Why fencing tokens?

## Related concepts
- [[Redis]]
- [[Idempotency]]
- [[Rate-Limiting-Design]]
