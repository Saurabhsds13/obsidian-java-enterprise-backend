---
type: concept
domain: database
topic: transactions
difficulty: hard
status: inbox
tags: [database]
---

# ACID

## Definition
The four guarantees of a reliable transaction: **Atomicity** (all-or-nothing), **Consistency** (invariants preserved), **Isolation** (concurrent transactions don't corrupt each other), **Durability** (committed data survives crashes). ACID is why relational databases are trusted for money and critical state.

## Why it matters
Understanding *how* each property is implemented — WAL, MVCC, isolation levels — separates "I use transactions" from "I know what the database guarantees and what I still have to handle myself" (e.g. Read Committed doesn't stop lost updates).

## How it works — the mechanism per property

### Atomicity — undo/rollback
Implemented with an **undo log** (or MVCC old versions). Mid-transaction changes can be rolled back; on crash, uncommitted work is undone during recovery.

### Consistency — constraints + app invariants
The DB enforces declared constraints (PK, FK, unique, check); the *application* is responsible for domain invariants. Consistency is the property the app and DB uphold *together* — the DB won't invent a valid state for you.

### Isolation — via [[Isolation-Levels]]
Concurrency control (locking or **MVCC**) provides the chosen level. Note the default (Read Committed) still permits lost updates — isolation is a **dial**, not a single guarantee.

### Durability — WAL (write-ahead log)
Before a change is applied to data files, it's written to the **WAL** and `fsync`ed. On crash, replay the WAL to recover committed transactions. `fsync` durability vs OS/disk caching is the real-world caveat (and the cost knob: group commit, `synchronous_commit`).

## Enterprise example
```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;   -- atomic pair
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
  INSERT INTO ledger(...) VALUES (...);                        -- audit in same tx
COMMIT;   -- durable + atomic: all three or none
```

## ACID vs BASE (distributed reality)
Single-node relational = ACID. Distributed systems often relax to **BASE** (Basically Available, Soft state, Eventual consistency — see [[Eventual-Consistency]]) because strict cross-node ACID needs coordination (2PC) that hurts availability/latency ([[CAP-Theorem]]). Enterprise pattern: keep money/state ACID in one datastore; use the **outbox/saga** ([[Message-Queues]]) for cross-service consistency rather than distributed transactions.

## Trade-offs
- Strong guarantees cost coordination (locks, fsync, aborts). Short transactions minimize the cost; distributed ACID (2PC) is often avoided for its latency/availability penalty.

## Common mistakes (senior-level)
- Assuming isolation prevents all anomalies at every level (Read Committed allows lost updates — see [[Isolation-Levels]]).
- Long transactions holding locks/versions → contention, [[Deadlocks]], MVCC bloat.
- Expecting cross-service atomicity from local transactions (need saga/outbox).
- Trusting durability without understanding `fsync`/replication settings.

## Interview questions (staff+)
- Explain each ACID property and how it's implemented (undo log, MVCC, WAL).
- Which property does the isolation level control, and why isn't it a single guarantee?
- ACID vs BASE — when do you relax which, and how do you keep money correct?
- How is durability achieved and what's the WAL/fsync trade-off?

## Related concepts
- [[Isolation-Levels]]
- [[Database-Transactions]]
- [[Deadlocks]]
- [[Eventual-Consistency]]
