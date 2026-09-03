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
The dominant [[JPA]] implementation — an ORM that maps entities to tables and manages the **Session** (JPA's persistence context), generating SQL, tracking changes via dirty checking, coordinating flush timing, and providing two cache layers. Understanding its runtime behavior is what turns JPA from "magic" into a predictable tool.

## Why it matters
Nearly every performance and correctness incident in a JPA app traces to Hibernate specifics: flush order, lazy proxies, cascade/orphan semantics, and cache staleness. Interviewers want to see you reason about the *generated SQL* and *when* it runs.

## How it works — the mechanism

### Session = persistence context
The `Session` holds managed entities, their load-time snapshots (for dirty checking), and a queue of pending actions. It provides the first-level cache and identity guarantee (see [[JPA]]).

### Flush and action ordering
On flush, Hibernate does **not** execute SQL in the order you called methods. It orders by operation type: **inserts → updates → deletes** (within the ordering, respecting FK dependencies). This surprises people who expect `delete` then `insert` and hit a unique-constraint violation. Control it with an explicit `flush()`, reordering, or a native statement when necessary. Default `FlushMode.AUTO` flushes before affected queries and at commit.

### Lazy loading via proxies
A lazy `@ManyToOne` is a **proxy subclass**; a lazy collection is a `PersistentCollection` wrapper. First access triggers a SELECT — but only while the Session is open. After the transaction closes the proxy is uninitialized → `LazyInitializationException`. Never "fix" this with `EAGER` (creates [[Hibernate-N-Plus-One|N+1]] and giant joins) — fetch deliberately.

### Cache layers
| Cache | Scope | Always on? | Notes |
|-------|-------|-----------|-------|
| First-level | Session/transaction | yes | identity + dirty checking |
| Second-level | SessionFactory (across tx) | opt-in | entities/collections; needs a provider (Ehcache/Infinispan); watch invalidation |
| Query cache | with L2 | opt-in | caches query result *ids*; easily stale — use sparingly |

## Enterprise example — avoid LazyInitializationException by projecting in-tx
```java
@Transactional(readOnly = true)
public OrderView load(Long id) {
    Order o = orders.findById(id).orElseThrow();
    // fetch what the view needs WHILE the session is open:
    return new OrderView(o.getId(),
                         o.getCustomer().getName(),          // triggers lazy load here, safely
                         o.getItems().stream().map(OrderItem::sku).toList());
}
```

## Cascade & orphanRemoval (a correctness minefield)
- `cascade = ALL` propagates persist/merge/remove to children; `orphanRemoval = true` deletes a child removed from the collection.
- `CascadeType.REMOVE` + a large collection can silently issue hundreds of deletes. Be intentional; don't cascade across aggregate boundaries.

## Bulk / batch performance
- Enable JDBC batching (`hibernate.jdbc.batch_size`), but **`GenerationType.IDENTITY` disables insert batching** (Hibernate needs the id immediately) — use `SEQUENCE` with a pooled optimizer for bulk inserts.
- For large updates/deletes, prefer bulk JPQL/HQL (`update ... where`) over loading-then-mutating N entities.
- Watch the **open-session-in-view** anti-pattern (Boot enables OSIV by default): it keeps the session open through view rendering to dodge `LazyInitializationException`, but hides N+1 and holds a connection longer — disable it and fetch explicitly.

## Trade-offs
- Powerful and productive, but leaky: performance depends on understanding generated SQL, flush ordering, and fetch strategy. It optimizes for developer velocity, not for hand-tuned SQL.

## Common mistakes (senior-level)
- `EAGER` everywhere to dodge `LazyInitializationException` → N+1 + cartesian joins.
- Relying on OSIV instead of fetching intentionally.
- Expecting method-call order to equal SQL order (flush reorders).
- Cascading REMOVE across aggregates; unbounded orphanRemoval.
- IDENTITY ids in a bulk-insert hot path (kills batching).

## Interview questions (staff+)
- What is the Session and how does dirty checking + flush ordering work?
- Why does `EAGER` cause N+1, and how do you fetch correctly?
- First-level vs second-level vs query cache, and when each is dangerous.
- What is open-session-in-view and why disable it?
- Why does IDENTITY generation prevent batch inserts?

## Related concepts
- [[JPA]]
- [[Hibernate-N-Plus-One]]
- [[Transactional]]
- [[Isolation-Levels]]
- [[Connection-Pooling]]
