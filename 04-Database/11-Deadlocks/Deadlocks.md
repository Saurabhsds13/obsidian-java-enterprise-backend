---
type: concept
domain: database
topic: locking
difficulty: hard
status: inbox
tags: [database]
---

# Deadlocks

## Definition
A cycle in the wait-for graph: two or more transactions each hold a lock the other needs, so none can proceed. The database detects the cycle and aborts a **victim** (rolling it back) to break it.

## Why it matters
Deadlocks cause transaction aborts, latency spikes, and user-visible errors under concurrency. The senior skill is knowing they're usually preventable with **consistent lock ordering**, and that the app must **retry** the victim.

## How it works — the mechanism
```text
Tx A: locks row 1 ...................... waits for row 2
Tx B: locks row 2 ...................... waits for row 1
        -> wait-for cycle -> DB deadlock detector aborts one (e.g. PG 40P01 / MySQL 1213)
```
- The engine runs a **deadlock detector** (cycle detection in the wait-for graph, or a lock-wait timeout) and picks a victim (often the one with least work done / fewest locks).
- Common real cause: two code paths update the **same rows in different orders**, or an index/gap lock interaction under Repeatable Read.

## Prevention (in priority order)
1. **Consistent lock ordering** — always acquire rows/tables in the same canonical order everywhere (e.g. lock account ids ascending). Eliminates the AB/BA cycle. This is the #1 fix.
2. **Keep transactions short** ([[Database-Transactions]]) — less time holding locks = smaller collision window.
3. **Lower the lock footprint** — precise `WHERE` on indexed columns (unindexed updates can lock more rows/gaps); appropriate [[Isolation-Levels]].
4. **Prefer optimistic locking** (`@Version`) over long-held pessimistic locks where contention is low.
5. **Retry the victim** — deadlocks are expected under load; wrap the transaction in bounded retry with backoff ([[Retry]]).

## Enterprise example — ordering + retry
```java
// Canonical ordering: always lock the lower id first -> no AB/BA cycle
long first = Math.min(from, to), second = Math.max(from, to);
retryOnDeadlock(3, () -> tx(() -> {
    accounts.lockAndDebit(first == from ? from : to, ...);
    accounts.lockAndCredit(...);
}));
```

## Deadlock vs lock wait vs livelock
- **Deadlock**: cyclic wait → detector aborts a victim.
- **Lock wait / blocking**: A simply waits for B to commit (no cycle) → resolved by a `lock_timeout`, not an abort.
- **Livelock**: transactions keep aborting and retrying in lockstep → add jitter to retry backoff.

## Trade-offs
- Pessimistic locks avoid conflicts but cut concurrency and invite deadlocks; optimistic locking scales but retries on conflict.
- Lower isolation reduces some lock contention but permits more anomalies.

## Common mistakes (senior-level)
- Inconsistent update order across code paths (the classic cause).
- No retry on the deadlock victim → user sees an error for a transient, expected event.
- Long transactions widening the collision window.
- Unindexed `UPDATE ... WHERE` locking far more rows than expected (gap/next-key locks under RR).

## Interview questions (staff+)
- What causes a deadlock and how does the DB resolve it?
- What's the single most effective prevention, and why?
- Deadlock vs lock-wait timeout vs livelock.
- Optimistic vs pessimistic locking for a hot-row update.
- Why can an unindexed UPDATE increase deadlock risk?

## Related concepts
- [[Database-Transactions]]
- [[Isolation-Levels]]
- [[ACID]]
- [[Retry]]
