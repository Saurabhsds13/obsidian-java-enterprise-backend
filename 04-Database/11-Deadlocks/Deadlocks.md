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
A cycle where two or more transactions each hold a lock the other needs, so none can proceed.

## Why it matters
Deadlocks cause transaction aborts and latency spikes in production. Understanding lock ordering prevents most of them.

## How it works
```text
Tx A: lock row 1 ... wants row 2
Tx B: lock row 2 ... wants row 1
-> cycle -> DB detects it and aborts one (deadlock victim)
```
Databases detect cycles and roll back a victim (e.g. Postgres error 40P01). The application should catch and retry the aborted transaction.

## Prevention
- **Consistent lock ordering**: always acquire locks (rows/tables) in the same order across the codebase.
- Keep transactions short ([[Database-Transactions]]).
- Reduce isolation contention; use appropriate [[Isolation-Levels]].
- For lost-update races, prefer optimistic locking (version column) over long-held pessimistic locks.

## Production usage
Log the deadlock graph, identify the conflicting statements, enforce ordering, and add bounded retry with backoff ([[Retry]]).

## Trade-offs
- Pessimistic locks avoid conflicts but reduce concurrency; optimistic locking scales but retries on conflict.

## Common mistakes
- Inconsistent update order across code paths.
- No retry on deadlock victims.

## Interview questions
- What causes a deadlock and how do you prevent it?
- Optimistic vs pessimistic locking?

## Related concepts
- [[Database-Transactions]]
- [[Isolation-Levels]]
- [[ACID]]
