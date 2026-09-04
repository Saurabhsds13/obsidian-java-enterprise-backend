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
A mechanism ensuring that, across multiple processes/machines, at most one holder executes a critical section at a time. Intrinsic JVM locks ([[synchronized]]) only coordinate threads *within one process*; distributed locks coordinate across a fleet.

## Why it matters
They're needed for leader-only jobs, preventing double-processing, and mutual exclusion on a shared resource — but they're famously subtle under failures (GC pauses, clock skew, expiry). The senior insight is usually: **avoid the lock** via idempotency or partitioning, and if you must use one, understand its failure modes.

## How it works — the mechanism

### Redis-based (simple, fast)
```text
SET lock:job42 <randomToken> NX PX 10000     # acquire iff absent, 10s TTL
# ... do work (must finish well under the TTL) ...
# release ONLY if token matches (atomic compare-and-delete via Lua):
EVAL "if redis.call('get',KEYS[1])==ARGV[1] then return redis.call('del',KEYS[1]) end" 1 lock:job42 <token>
```
- **TTL** prevents a permanent lock if the holder crashes.
- **Random token + compare-and-delete** prevents deleting *someone else's* lock (yours may have expired and been re-acquired).

### The safety gap: GC pause / clock skew
If the holder stalls (long GC, VM pause) past the TTL, the lock expires, another node acquires it, and now **two** nodes act. TTL alone can't prevent this. The fix is a **fencing token**: a monotonically increasing number issued at acquire time; the *protected resource* rejects any write with a token lower than the highest it has seen. This makes a stale holder's late write harmless.

### Consensus-based (stronger)
ZooKeeper/etcd use ephemeral sequential nodes + consensus (ZAB/Raft) — the lock auto-releases when the session dies, and ordering is linearizable. Stronger correctness than single-node Redis, at higher latency/complexity. (**Redlock** across N Redis nodes is debated — many argue it still doesn't guarantee correctness without fencing.)

## Prefer these alternatives first
| Instead of a lock | Technique |
|-------------------|-----------|
| Prevent duplicate processing | **Idempotency** key / dedup store ([[Idempotency]]) |
| Serialize work per entity | **Partition** by key ([[Kafka]] partitions → one consumer per key) |
| At-most-one insert | **DB unique constraint** |
| Leader-only task | **Leader election** (etcd/ZooKeeper) |

## Enterprise example — leader-only scheduled job
```java
// Only the instance holding the lock runs the nightly reconciliation
if (lock.tryAcquire("recon:nightly", Duration.ofMinutes(30))) {
    try { reconciliationService.run(); }   // must be < TTL; make it idempotent anyway
    finally { lock.release("recon:nightly", token); }
}
```

## Trade-offs
- Redis lock: simple, low-latency, but weak under partitions/pauses without fencing.
- Consensus lock: correct and auto-releasing, but higher latency and an extra dependency.
- Any lock reduces concurrency and adds a failure dependency — use sparingly, keep the section short.

## Common mistakes (senior-level)
- Deleting a lock you no longer own (no token check).
- Relying on TTL for correctness without a **fencing token** (GC-pause double-execution).
- Long critical sections that outlive the TTL.
- Using a distributed lock where idempotency/partitioning/DB-constraint would be simpler and safer.
- Trusting Redlock for strict correctness without fencing.

## Interview questions (staff+)
- Design a Redis lock; walk the acquire/release and why the token + Lua matter.
- Why does TTL alone not guarantee mutual exclusion, and how do fencing tokens fix it?
- When would you use ZooKeeper/etcd over Redis?
- Name three ways to avoid needing a distributed lock at all.

## Related concepts
- [[Redis]]
- [[Idempotency]]
- [[synchronized]]
- [[Rate-Limiting-Design]]
