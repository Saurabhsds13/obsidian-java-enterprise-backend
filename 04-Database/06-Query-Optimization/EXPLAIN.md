---
type: concept
domain: database
topic: query-optimization
difficulty: medium
status: inbox
tags: [database]
---

# EXPLAIN

## Definition
A command that shows the database's execution plan for a query — how it will access tables, which indexes it uses, join methods, and estimated costs.

## Why it matters
It's the evidence-based way to diagnose slow queries: measure before optimizing.

## How it works
- `EXPLAIN` shows the planned strategy; `EXPLAIN ANALYZE` actually runs it and reports real timings and row counts.
- Look for: **Seq Scan** on large tables (missing index?), estimated vs actual rows (bad statistics), and expensive join methods.

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 50;
-- Want: Index Scan using idx_orders_cust_created ...
```

## Production usage
When a query is slow, run `EXPLAIN ANALYZE`, find the costly node, add/adjust an [[Indexes|index]] or rewrite the query, then re-check. Keep table statistics fresh (`ANALYZE`).

## Trade-offs
- `EXPLAIN ANALYZE` executes the query — be careful with writes (wrap in a transaction and roll back).

## Common mistakes
- Optimizing by guesswork instead of reading the plan.
- Ignoring the estimated-vs-actual row gap (stale stats).

## Interview questions
- How do you diagnose a slow query? (see [[How-would-you-handle-a-database-that-becomes-the-bottleneck]])
- Seq scan vs index scan — when is each fine?

## Related concepts
- [[Indexes]]
- [[Composite-Index]]
- [[Query-Optimization]]
