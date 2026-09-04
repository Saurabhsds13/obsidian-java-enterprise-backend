---
type: concept
domain: database
topic: transactions
difficulty: medium
status: inbox
tags: [database]
---

# Database Transactions

## Definition
A unit of work executed atomically against the database — it either commits entirely or rolls back entirely — with the guarantees defined by [[ACID]] and the concurrency behavior set by [[Isolation-Levels]]. It is the foundation Spring's [[Transactional]] builds on.

## Why it matters
Transactions protect multi-statement invariants (debit + credit + audit). The senior concern is the **cost of holding one open**: locks, MVCC versions, and a pooled connection are all tied up for the transaction's whole duration — long transactions are a top production incident cause.

## How it works — the mechanism
```sql
BEGIN;
  INSERT INTO orders (...) VALUES (...);
  UPDATE inventory SET qty = qty - 1 WHERE sku = 'ABC' AND qty >= 1;
COMMIT;   -- or ROLLBACK
```
- **Boundaries**: `BEGIN … COMMIT/ROLLBACK`. Everything between shares one snapshot/lock scope.
- **Locks + versions held until the transaction ends** — the longer it runs, the more contention it causes and the longer MVCC old versions can't be vacuumed.
- **Savepoints** allow partial rollback within a transaction (the DB primitive behind `NESTED` propagation).

## The cardinal rule: keep transactions short and DB-only
Never hold a transaction open across a network/HTTP call or slow computation:
- It pins a pooled connection ([[Connection-Pooling]]) → pool exhaustion under load.
- It holds row locks → [[Deadlocks]] and blocked writers.
- Pattern: do external I/O **before** or **after** the transaction; inside, only touch the DB.

## Enterprise example — compute outside, commit inside
```java
PricingResult priced = pricingClient.quote(cmd);   // remote call OUTSIDE any tx
txTemplate.executeWithoutResult(status -> {         // short DB-only tx
    orders.save(Order.of(cmd, priced));
    inventory.decrement(cmd.sku(), cmd.qty());
});
notifier.enqueue(cmd);                              // async, OUTSIDE the tx
```

## Trade-offs
- Strong atomicity/consistency vs concurrency: broader/longer transactions serialize more work and hold more resources.
- Read-only transactions (`readOnly`) let the engine/replica optimize and skip write bookkeeping.

## Common mistakes (senior-level)
- Remote calls / long computation inside a transaction (pool exhaustion + lock contention).
- Assuming a `@Transactional` method is transactional when self-invoked (proxy bypass — see [[Why-does-Transactional-sometimes-not-work]]).
- Read-modify-write across statements without a version/lock (lost update — [[Isolation-Levels]]).
- Giant "God transactions" spanning many aggregates.

## Interview questions (staff+)
- What exactly is held for a transaction's duration, and why does that cap transaction length?
- Why must you never call an external API inside a transaction?
- What are savepoints and how do they relate to nested transactions?
- How do you structure a flow that needs both a remote call and a DB write?

## Related concepts
- [[ACID]]
- [[Isolation-Levels]]
- [[Deadlocks]]
- [[Transactional]]
- [[Connection-Pooling]]
