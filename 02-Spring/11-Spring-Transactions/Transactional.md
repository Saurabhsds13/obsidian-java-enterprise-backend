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
A declarative marker that wraps a method in a database transaction, committing on normal return and rolling back on runtime exceptions.

## Why it matters
It's how Spring guarantees atomicity across multiple data operations. Its proxy-based mechanism has famous gotchas (self-invocation, checked exceptions) that appear constantly in interviews and production.

## How it works
- Backed by [[Spring-AOP]]: Spring wraps the bean in a **proxy** that begins/commits/rolls back around the method.
- **Propagation** controls how nested transactional calls behave (`REQUIRED` default, `REQUIRES_NEW`, `NESTED`...).
- **Isolation** maps to DB [[Isolation-Levels]].
- Default rollback is on unchecked (`RuntimeException`) only; checked exceptions do **not** roll back unless you set `rollbackFor`.

```java
@Service
public class TransferService {
    @Transactional
    public void transfer(Long from, Long to, BigDecimal amt) {
        accounts.debit(from, amt);
        accounts.credit(to, amt);   // both commit or both roll back
    }
}
```

## Why it "sometimes doesn't work"
- **Self-invocation**: calling a `@Transactional` method from another method in the *same* bean bypasses the proxy (no transaction). Move it to another bean or use self-injection.
- **Non-public methods**: proxies apply to public methods by default.
- **Checked exception thrown**: commits unless `rollbackFor` is set.

## Production usage
Keep transactions short; never do remote calls or long I/O inside them. Use `readOnly = true` for queries. See [[Why-does-Transactional-sometimes-not-work]].

## Trade-offs
- Declarative simplicity vs hidden proxy semantics that surprise developers.

## Common mistakes
- Self-invocation; expecting rollback on checked exceptions; long transactions holding locks ([[Deadlocks]]).

## Interview questions
- Why does `@Transactional` sometimes not work?
- Explain propagation `REQUIRED` vs `REQUIRES_NEW`.

## Related concepts
- [[Spring-AOP]]
- [[Isolation-Levels]]
- [[Database-Transactions]]
- [[Hibernate]]
