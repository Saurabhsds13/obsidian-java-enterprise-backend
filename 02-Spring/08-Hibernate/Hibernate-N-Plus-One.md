---
type: concept
domain: spring
topic: hibernate
difficulty: hard
status: inbox
tags: [spring]
---

# Hibernate N+1 Problem

## Definition
A performance anti-pattern where fetching N parent rows triggers N additional queries to load each parent's lazy association — 1 + N queries instead of 1 or 2.

## Why it matters
It's the single most common ORM performance bug in production, turning a fast page into hundreds of queries.

## How it works
```java
List<Order> orders = repo.findAll();      // 1 query
for (Order o : orders) {
    o.getCustomer().getName();            // +1 query per order (N queries)
}
```
Each lazy `getCustomer()` fires its own SELECT.

## Fixes
- **Fetch join**: `@Query("select o from Order o join fetch o.customer")` — one query.
- **Entity graph**: `@EntityGraph(attributePaths = "customer")` on the repository method.
- **Batch fetching**: `@BatchSize(size = 50)` / `hibernate.default_batch_fetch_size` to load associations in batches.

## Production usage
Detect it by logging SQL (`spring.jpa.show-sql` in dev, or statement metrics) and watching for repeated identical queries. Choose fetch joins for known access paths.

## Trade-offs
- Fetch joins can cause a cartesian product with multiple collections — fetch one collection at a time.

## Common mistakes
- "Solving" it by switching to `EAGER` globally (creates N+1 elsewhere and huge joins).

## Interview questions
- What causes the N+1 problem and how do you fix it? (see [[What-causes-the-N-plus-1-problem]])
- Fetch join vs entity graph vs batch size?

## Related concepts
- [[Hibernate]]
- [[JPA]]
- [[Query-Optimization]]
