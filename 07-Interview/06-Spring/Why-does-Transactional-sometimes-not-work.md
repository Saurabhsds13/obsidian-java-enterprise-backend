---
type: interview
domain: spring
topic: transactions
difficulty: hard
status: inbox
tags: [interview, spring]
---

# Why does @Transactional sometimes not work?

## Question
> A method is annotated `@Transactional` but no transaction seems to apply. Why?

## Short answer
Because it's proxy-based: self-invocation, non-public methods, or throwing checked exceptions all bypass or skip the expected behavior.

## Detailed answer
Spring wraps the bean in a [[Spring-AOP|proxy]]. The transactional logic runs only when the call goes **through** the proxy.
- **Self-invocation**: `this.method()` from another method in the same bean skips the proxy → no transaction. Fix: move it to another bean, or self-inject.
- **Non-public method**: proxies apply to public methods by default.
- **Checked exception**: default rollback is on `RuntimeException` only; a checked exception commits unless `rollbackFor` is set.
- **Wrong propagation** or a swallowed exception can also hide expected rollback.

## Example
```java
@Service
class OrderService {
    public void place() { this.charge(); }     // self-call -> no tx around charge()
    @Transactional public void charge() { ... }
}
```

## Production relevance
Keep transactional methods public and called across bean boundaries; set `rollbackFor` when using checked exceptions; keep transactions short.

## Common mistake
Assuming the annotation alone guarantees a transaction regardless of how the method is called.

## Follow-up questions
- Explain propagation `REQUIRED` vs `REQUIRES_NEW`.
- How would you fix self-invocation?

## Related concepts
- [[Transactional]]
- [[Spring-AOP]]
- [[Isolation-Levels]]
