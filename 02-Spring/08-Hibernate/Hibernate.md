---
type: concept
domain: spring
topic: hibernate
difficulty: hard
status: inbox
tags: [spring]
---

# Hibernate

## Definition
The most widely used [[JPA]] implementation — an ORM that maps Java objects to relational tables and manages persistence, caching, and query generation.

## Why it matters
Hibernate's behavior (lazy loading, flush timing, caching) drives real query patterns and performance. Treating it as a black box leads to N+1 storms and stale-data bugs.

## How it works
- **Session** = the JPA persistence context; tracks managed entities and does **dirty checking** at flush/commit.
- **Lazy loading** via proxies: associations load on first access — but only while the session is open.
- **Caches**: first-level (session-scoped, always on); optional second-level (shared across sessions).
- Generates SQL from JPQL/criteria; you can drop to native SQL when needed.

```java
@Transactional(readOnly = true)
public OrderView load(Long id) {
    Order order = repo.findById(id).orElseThrow();
    order.getItems().size();   // triggers lazy load inside the transaction
    return OrderView.from(order);
}
```

## Production usage
Keep entity graphs lazy; fetch deliberately with `join fetch` or `@EntityGraph`. Map to DTOs inside the transaction to avoid `LazyInitializationException`. See [[Hibernate-N-Plus-One]].

## Trade-offs
- Powerful but leaky: performance depends on understanding generated SQL and flush semantics.

## Common mistakes
- Accessing lazy associations after the transaction closes (`LazyInitializationException`).
- Using `EAGER` to "fix" it (creates N+1 and huge joins).

## Interview questions
- How does lazy loading work and when does it fail?
- First-level vs second-level cache?

## Related concepts
- [[JPA]]
- [[Hibernate-N-Plus-One]]
- [[Transactional]]
- [[Isolation-Levels]]
