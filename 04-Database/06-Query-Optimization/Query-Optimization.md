---
type: concept
domain: database
topic: query-optimization
difficulty: hard
status: inbox
tags: [database]
---

# Query Optimization

## Definition
The practice of making queries faster by improving indexes, query structure, schema, and statistics — guided by execution plans.

## Why it matters
The database is the most common backend bottleneck. Query tuning often yields larger wins than adding hardware.

## How it works
A disciplined loop:
1. **Measure**: find slow queries (slow-query log, metrics) and read the plan ([[EXPLAIN]]).
2. **Index**: add/adjust [[Indexes]] and [[Composite-Index|composite indexes]] for the query shape.
3. **Rewrite**: avoid `SELECT *`, functions on indexed columns, and unnecessary `DISTINCT`/subqueries; use appropriate [[SQL-Joins]].
4. **Reduce work**: paginate ([[Pagination]] via keyset), fetch only needed columns, batch to avoid [[Hibernate-N-Plus-One|N+1]].
5. **Maintain**: keep statistics current; watch [[Connection-Pooling|connection pool]] health.

## Production usage
Tune the top offenders by total time (frequency × latency), not just the single slowest query.

## Trade-offs
- Indexes speed reads but slow writes; denormalization speeds reads but risks consistency.

## Common mistakes
- Optimizing without measuring.
- Deep offset pagination and `SELECT *` on wide tables.

## Interview questions
- How would you handle a database that becomes the bottleneck? (see [[How-would-you-handle-a-database-that-becomes-the-bottleneck]])
- Walk through tuning a slow query.

## Related concepts
- [[EXPLAIN]]
- [[Indexes]]
- [[Connection-Pooling]]
- [[Hibernate-N-Plus-One]]
