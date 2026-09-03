---
type: concept
domain: spring
topic: jpa
difficulty: hard
status: inbox
tags: [spring]
---

# JPA

## Definition
Jakarta Persistence API — a specification for ORM in Java. Its central runtime abstraction is the **persistence context**: a transaction-scoped, first-level cache of *managed* entities that the `EntityManager` tracks, dirty-checks, and flushes to the database. [[Hibernate]] is the dominant implementation; Spring Data JPA layers repositories on top.

## Why it matters
JPA's power *and* its production landmines both come from the persistence context being implicit: dirty checking, flush timing, lazy loading, and identity all behave in ways that surprise people who think of it as "SQL with annotations." Architect-level questions live in the entity lifecycle and flush semantics.

## How it works — the mechanism

### Entity lifecycle (state machine)
```
new Order()            -> TRANSIENT  (no id, not tracked)
em.persist(o)          -> MANAGED    (tracked, in persistence context)
tx commit / em close   -> DETACHED   (has id, no longer tracked)
em.remove(o)           -> REMOVED    (scheduled for delete)
em.merge(detached)     -> returns a MANAGED copy (does NOT re-attach the argument)
```

### The persistence context (first-level cache)
- Scoped to the transaction (in Spring, one per `@Transactional` boundary — see [[Transactional]]).
- **Identity guarantee**: within one context, `findById(1)` twice returns the *same* instance (`==`).
- **Dirty checking**: on flush, Hibernate compares each managed entity to a snapshot taken at load and issues `UPDATE`s for changed fields — **you don't call save() to update a managed entity**.
- **Flush** (sending pending SQL) happens automatically before a query that might be affected, and at commit. Flush ≠ commit.

### Repository update without save()
```java
@Transactional
public void applyDiscount(Long id, BigDecimal pct) {
    Order o = orders.findById(id).orElseThrow();  // MANAGED
    o.setTotal(o.getTotal().multiply(pct));        // just mutate...
    // no repo.save() needed — dirty checking flushes an UPDATE at commit
}
```

## Fetch strategy is where performance is won or lost
- `@ManyToOne`/`@OneToOne` default to **EAGER**; `@OneToMany`/`@ManyToMany` default to **LAZY**. The usual advice: **make everything LAZY** and fetch deliberately per query.
- Access a lazy association *after* the context closes → `LazyInitializationException`. Solve by fetching inside the transaction and mapping to DTOs, or with fetch joins / `@EntityGraph` (see [[Hibernate-N-Plus-One]]).

## Locking (concurrency control)
| Strategy | Mechanism | Use when |
|----------|-----------|----------|
| **Optimistic** (`@Version`) | version column; on write, `WHERE version = ?`; mismatch → `OptimisticLockException` | low contention, high throughput (default choice) |
| **Pessimistic** (`PESSIMISTIC_WRITE`) | DB row lock `SELECT ... FOR UPDATE` | short critical sections with high contention |

Optimistic locking is how you prevent lost updates without holding DB locks — a key scalability decision (pairs with [[Isolation-Levels]]).

## Enterprise example
```java
@Entity
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) Long id;
    @Version long version;                                   // optimistic lock
    @ManyToOne(fetch = FetchType.LAZY) Customer customer;    // lazy on purpose
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    List<OrderItem> items = new ArrayList<>();
}
```

## Trade-offs
- Advantages: productivity, identity/caching, portability, dirty checking.
- Disadvantages: hidden SQL and flush timing; leaky abstraction (you still must understand generated queries); easy to create N+1 and `LazyInitializationException`.

## Common mistakes (senior-level)
- Returning entities from controllers → lazy access outside the tx (`LazyInitializationException`) and serializing proxies. Map to DTOs inside the transaction.
- Calling `save()` expecting it to matter for a managed entity (the update already happens via dirty checking; `save` on a managed entity is a no-op merge).
- Using `GenerationType.IDENTITY` when you need JDBC batch inserts (IDENTITY disables batching — prefer `SEQUENCE` with a pooled allocator for bulk).
- `equals`/`hashCode` on a generated `@Id` that's null before persist (breaks Set membership — use a business key).

## Interview questions (staff+)
- Describe the entity lifecycle states and the transitions between them.
- What is dirty checking and why don't you need `save()` to update a managed entity?
- Flush vs commit — when does Hibernate flush?
- Optimistic vs pessimistic locking — which by default and why?
- Why does `merge` not re-attach its argument?

## Related concepts
- [[Hibernate]]
- [[Hibernate-N-Plus-One]]
- [[Transactional]]
- [[Database-Transactions]]
- [[Isolation-Levels]]
