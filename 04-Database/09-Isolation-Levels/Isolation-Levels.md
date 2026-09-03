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
Settings that control how much concurrent transactions can affect each other, trading consistency against concurrency.

## Why it matters
The isolation level decides which anomalies are possible. Picking the wrong one causes subtle data bugs or unnecessary contention.

## How it works
Levels (weakest → strongest) and the anomalies they allow:

| Level | Dirty read | Non-repeatable read | Phantom read |
|-------|-----------|---------------------|--------------|
| Read Uncommitted | possible | possible | possible |
| Read Committed | no | possible | possible |
| Repeatable Read | no | no | possible* |
| Serializable | no | no | no |

*PostgreSQL's Repeatable Read (snapshot) also prevents phantoms in practice.

- **Dirty read**: reading uncommitted data.
- **Non-repeatable read**: same row read twice returns different values.
- **Phantom read**: same query returns new rows on re-run.

## Production usage
Read Committed is the common default (PostgreSQL). Use Repeatable Read/Serializable for invariants that must hold across multiple reads. For lost-update prevention, use optimistic locking (version column) or `SELECT ... FOR UPDATE`.

## Trade-offs
- Higher isolation = fewer anomalies but more locking/aborts and lower concurrency.

## Common mistakes
- Assuming the default prevents lost updates.
- Using Serializable everywhere and creating contention.

## Interview questions
- Explain the isolation levels and the anomalies each prevents.
- How do you prevent lost updates?

## Related concepts
- [[ACID]]
- [[Deadlocks]]
- [[Transactional]]
- [[Database-Transactions]]
