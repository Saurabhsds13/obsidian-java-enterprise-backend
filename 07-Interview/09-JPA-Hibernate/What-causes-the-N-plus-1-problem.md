---
type: interview
domain: spring
topic: hibernate
difficulty: hard
status: inbox
tags: [interview, spring]
---

# What causes the N+1 problem?

## Question
> What is the N+1 query problem in JPA/Hibernate and how do you fix it?

## Short answer
Loading N parents then lazily loading each parent's association fires 1 + N queries. Fix with fetch joins, entity graphs, or batch fetching.

## Detailed answer
With lazy associations, iterating N entities and touching a lazy field triggers one query per entity. Fixes: `join fetch` (one query), `@EntityGraph` (declarative fetch), or `@BatchSize`/`default_batch_fetch_size` (load associations in batches). Detect it by logging SQL and spotting repeated identical queries.

## Example
```java
@Query("select o from Order o join fetch o.customer")
List<Order> findAllWithCustomer();
```

## Production relevance
The most common ORM performance bug; catch it in load tests and SQL metrics.

## Common mistake
"Fixing" it by switching everything to `EAGER`, which creates N+1 elsewhere and huge joins.

## Follow-up questions
- Fetch join vs entity graph vs batch size?
- Why can fetch-joining two collections cause a cartesian product?

## Related concepts
- [[Hibernate-N-Plus-One]]
- [[Hibernate]]
- [[Query-Optimization]]
