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
A unit of work executed against the database that is treated atomically: it either commits entirely or rolls back entirely.

## Why it matters
Transactions protect invariants across multiple statements (e.g. debit + credit). They're the foundation Spring's [[Transactional]] builds on.

## How it works
```sql
BEGIN;
  INSERT INTO orders (...) VALUES (...);
  UPDATE inventory SET qty = qty - 1 WHERE sku = 'ABC';
COMMIT;   -- or ROLLBACK on failure
```
- Boundaries: `BEGIN` … `COMMIT`/`ROLLBACK`.
- Concurrency behavior governed by [[Isolation-Levels]]; guarantees defined by [[ACID]].
- Locks are held for the transaction's duration → long transactions increase contention and [[Deadlocks]].

## Production usage
Keep transactions short and free of remote calls/long I/O. In Spring, prefer declarative [[Transactional]] and mark read paths `readOnly`.

## Trade-offs
- Strong consistency vs concurrency: longer/broader transactions serialize more work.

## Common mistakes
- Doing HTTP calls inside a transaction (holds locks/connections).
- Assuming a `@Transactional` method is transactional when self-invoked (see [[Why-does-Transactional-sometimes-not-work]]).

## Interview questions
- What defines a transaction boundary?
- Why keep transactions short?

## Related concepts
- [[ACID]]
- [[Isolation-Levels]]
- [[Deadlocks]]
- [[Transactional]]
