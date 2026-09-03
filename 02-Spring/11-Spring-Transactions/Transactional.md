---
type: concept
domain: spring
topic: transactions
difficulty: hard
status: inbox
tags: [spring]
---

# @Transactional

## Definition
A declarative annotation that wraps a method in a database transaction via [[Spring-AOP]]. A `TransactionInterceptor` (around advice) uses a `PlatformTransactionManager` to begin/suspend/commit/rollback around the method according to its **propagation**, **isolation**, **rollback rules**, `readOnly`, and `timeout` attributes.

## Why it matters
It's the atomicity boundary for nearly all enterprise Java write paths, and its proxy-based nature produces the most famous Spring footguns (self-invocation, checked-exception commit). Architect-level questions go into propagation semantics, transaction-vs-connection lifecycle, and why you must never do remote I/O inside a transaction.

## How it works — the mechanism
1. The proxy's `TransactionInterceptor` fires before the method.
2. It asks the `TransactionManager` for a transaction per the **propagation** rule (may reuse, suspend, or start one).
3. On the current thread it binds a connection/`EntityManager` via `TransactionSynchronizationManager` (thread-bound resources — this is why a transaction is thread-confined and doesn't follow an `@Async` hop).
4. On normal return → commit (flush for JPA first). On a rollback-triggering exception → rollback.

### Propagation (the matrix worth knowing)
| Propagation | If a tx exists | If none exists |
|-------------|----------------|----------------|
| `REQUIRED` (default) | join it | start one |
| `REQUIRES_NEW` | **suspend** it, start a new independent tx | start one |
| `NESTED` | savepoint within the current tx | start one |
| `SUPPORTS` | join it | run non-transactionally |
| `MANDATORY` | join it | **throw** |
| `NEVER` | **throw** | run non-transactionally |
| `NOT_SUPPORTED` | suspend it, run non-transactionally | run non-transactionally |

Key insight: `REQUIRES_NEW` commits/rolls back **independently** (use for audit logs that must persist even if the outer tx rolls back). `NESTED` uses a savepoint — inner rollback doesn't kill the outer.

### Rollback rules (the trap)
Default rollback is on `RuntimeException` and `Error` only. A **checked exception commits** unless you declare `@Transactional(rollbackFor = ...)`. Also, once any participating method marks the tx rollback-only, an outer `REQUIRED` commit throws `UnexpectedRollbackException`.

## Why it "sometimes doesn't work"
- **Self-invocation**: `this.txMethod()` bypasses the proxy → no transaction (see [[Spring-AOP]]). Move to another bean or self-inject.
- **Non-public method**: proxy advises public methods by default.
- **Checked exception** thrown without `rollbackFor` → commits.
- **Swallowed exception** inside the method → no rollback signal.
- **Wrong manager** (JPA vs JDBC) or multiple datasources without the right qualifier.

## Enterprise example
```java
@Service
public class OrderService {
    @Transactional                                   // REQUIRED, rollback on RuntimeException
    public void placeOrder(CreateOrder cmd) {
        Order o = orders.save(Order.from(cmd));       // participates in the tx
        inventory.reserve(cmd.sku(), cmd.qty());      // same tx -> atomic with the save
        audit.record(o.id());                         // see REQUIRES_NEW below
        // DO NOT call an external HTTP API here — it holds the DB connection open
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)  // persists even if caller rolls back
    public void record(Long orderId) { auditRepo.save(new AuditEntry(orderId)); }
}
```

## Trade-offs
- Declarative simplicity vs hidden proxy semantics.
- `readOnly = true` lets Hibernate skip dirty-checking/flush and lets the DB/replica optimize — measurable win on query paths, but it's a hint, not a hard guarantee.
- Long transactions hold connections and locks → [[Connection-Pooling|pool exhaustion]] and [[Deadlocks]].

## Common mistakes (senior-level)
- Remote calls / long I/O inside a transaction (holds a pooled connection + row locks the whole time).
- Relying on rollback for checked exceptions without `rollbackFor`.
- Self-invocation; `@Transactional` on `private` methods.
- Expecting the transaction to propagate across an `@Async`/new thread (it's thread-bound).
- Catching-and-swallowing inside the tx, then wondering why it committed.

## Interview questions (staff+)
- Walk `REQUIRED` vs `REQUIRES_NEW` vs `NESTED` with a concrete audit-log example.
- Why does `@Transactional` sometimes not apply? Enumerate the causes.
- Default rollback rules — why does a checked exception commit?
- What does `readOnly = true` actually change?
- Why is calling an external API inside a transaction dangerous?

## Related concepts
- [[Spring-AOP]]
- [[Database-Transactions]]
- [[Isolation-Levels]]
- [[Hibernate]]
- [[Connection-Pooling]]
