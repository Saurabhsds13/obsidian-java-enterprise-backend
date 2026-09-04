---
type: concept
domain: database
topic: transactions
difficulty: hard
status: inbox
tags: [database]
---

# Isolation Levels

## Definition
The **I** in [[ACID]]: settings that control which concurrency **anomalies** are possible between overlapping transactions, trading consistency against concurrency. The SQL standard defines four levels by the anomalies they forbid; real engines implement them via locking or **MVCC**.

## Why it matters
The isolation level silently decides whether "read balance, then update" is safe. Picking wrong causes lost updates and phantom bugs that only appear under concurrency. Architect depth means knowing the anomalies, how MVCC differs from the standard, and how to prevent lost updates explicitly.

## How it works — the anomalies and levels
| Level | Dirty read | Non-repeatable read | Phantom | Lost update |
|-------|-----------|---------------------|---------|-------------|
| Read Uncommitted | possible | possible | possible | possible |
| **Read Committed** (PG default) | no | possible | possible | possible |
| **Repeatable Read** | no | no | possible* | prevented in snapshot impls |
| **Serializable** | no | no | no | no |

- **Dirty read**: seeing another tx's uncommitted data.
- **Non-repeatable read**: re-reading a row returns a different value (someone committed an update).
- **Phantom**: re-running a range query returns new rows (someone inserted).
- **Lost update**: two read-modify-write cycles clobber each other.

### MVCC (how modern engines actually do it)
PostgreSQL/InnoDB use **Multi-Version Concurrency Control**: writers create new row versions instead of blocking readers; each transaction reads a **snapshot** consistent as of its start (RR) or statement (RC). "Readers don't block writers, writers don't block readers." Consequences:
- PG **Repeatable Read = snapshot isolation**, which *also prevents phantoms* in practice (hence the `*` above) but can fail with a **serialization error** on write-write conflicts (`could not serialize access`) — the app must retry.
- PG **Serializable = SSI** (Serializable Snapshot Isolation): detects dangerous read/write dependency cycles and aborts a transaction rather than locking everything.
- MVCC's cost: old versions must be **vacuumed** (PG) / purged (InnoDB); long-running transactions block cleanup → **bloat**.

## Preventing lost updates (the practical question)
```sql
-- Option A: optimistic (app-level version column) — best for low contention
UPDATE accounts SET balance = balance - 100, version = version + 1
WHERE id = 42 AND version = :expected;     -- 0 rows updated => conflict => retry

-- Option B: pessimistic lock — short critical section, high contention
SELECT balance FROM accounts WHERE id = 42 FOR UPDATE;   -- row lock until commit

-- Option C: do the arithmetic in the DB atomically (no read-modify-write in the app)
UPDATE accounts SET balance = balance - 100 WHERE id = 42 AND balance >= 100;
```

## Trade-offs
- Higher isolation → fewer anomalies but more locking/aborts and lower concurrency.
- Read Committed (default) is fast but allows lost updates — you must guard multi-step invariants yourself.
- Serializable is safest but costs throughput and forces retry-on-abort logic.

## Common mistakes (senior-level)
- Assuming the default level prevents lost updates (Read Committed does not).
- Read-modify-write in application code without `@Version`/`FOR UPDATE`/atomic SQL.
- Using Serializable everywhere → contention and surprise serialization-failure exceptions with no retry logic.
- Long-running transactions on MVCC engines → version bloat / vacuum can't keep up.
- Not handling `could not serialize access` (must retry the whole transaction).

## Interview questions (staff+)
- Enumerate the anomalies and which level prevents each.
- How does MVCC serve reads without blocking writes, and what must be vacuumed?
- PostgreSQL Repeatable Read vs the SQL standard — why are phantoms prevented?
- Give three ways to prevent a lost update and when to use each.
- What is SSI (Serializable) and what does the app have to handle?

## Related concepts
- [[ACID]]
- [[Database-Transactions]]
- [[Deadlocks]]
- [[Transactional]]
