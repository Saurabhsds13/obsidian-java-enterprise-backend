---
type: concept
domain: spring
topic: jpa
difficulty: medium
status: inbox
tags: [spring]
---

# JPA

## Definition
Jakarta Persistence API — a specification for object-relational mapping in Java. [[Hibernate]] is the most common implementation; Spring Data JPA adds repositories on top.

## Why it matters
JPA is how most enterprise Java apps talk to relational databases. Its abstractions (persistence context, lazy loading) cause the majority of production data-access bugs when misunderstood.

## How it works
- **Entity**: a mapped class (`@Entity`).
- **EntityManager**: manages the **persistence context** (first-level cache) — a set of managed entities within a transaction.
- **Entity lifecycle**: transient → managed → detached → removed.
- **Dirty checking**: changes to managed entities are flushed automatically at transaction commit.

```java
@Entity
public class Order {
    @Id @GeneratedValue Long id;
    @ManyToOne(fetch = FetchType.LAZY) Customer customer;
}

public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerId(Long customerId);
}
```

## Production usage
Use lazy associations by default; fetch what you need with fetch joins/entity graphs to avoid the [[Hibernate-N-Plus-One|N+1 problem]]. Wrap write flows in [[Transactional]].

## Trade-offs
- Advantages: productivity, portability, caching.
- Disadvantages: hidden queries, lazy-loading pitfalls, leaky abstraction over SQL.

## Common mistakes
- Returning entities from controllers (triggers lazy loads outside a transaction).
- EAGER fetching everything.

## Interview questions
- What is the persistence context?
- Explain dirty checking.

## Related concepts
- [[Hibernate]]
- [[Hibernate-N-Plus-One]]
- [[Transactional]]
- [[Database-Transactions]]
