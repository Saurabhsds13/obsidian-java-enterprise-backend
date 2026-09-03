---
type: concept
domain: database
topic: transactions
difficulty: medium
status: inbox
tags: [database]
---

# ACID

## Definition
The four guarantees of a reliable transaction: **Atomicity**, **Consistency**, **Isolation**, **Durability**.

## Why it matters
ACID is why relational databases are trusted for money and critical state. Understanding each property clarifies what the DB does and does not protect.

## How it works
- **Atomicity**: all operations in a transaction succeed or none do (commit vs rollback).
- **Consistency**: a transaction moves the DB from one valid state to another (constraints hold).
- **Isolation**: concurrent transactions don't corrupt each other; controlled by [[Isolation-Levels]].
- **Durability**: once committed, data survives crashes (write-ahead log).

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;   -- atomic + durable; both or neither
```

## Production usage
Wrap multi-step invariants (transfers, order+inventory) in one transaction ([[Transactional]]). Keep transactions short to reduce lock contention and [[Deadlocks]].

## Trade-offs
- Strong guarantees cost coordination; distributed systems often trade strict ACID for availability/[[Eventual-Consistency]].

## Common mistakes
- Assuming isolation prevents all anomalies at every level (it doesn't — see [[Isolation-Levels]]).

## Interview questions
- Explain each ACID property.
- Which property does the isolation level control?

## Related concepts
- [[Isolation-Levels]]
- [[Database-Transactions]]
- [[Deadlocks]]
