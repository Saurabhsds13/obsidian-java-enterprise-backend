---
type: concept
domain: database
topic: sql
difficulty: medium
status: inbox
tags: [database]
---

# SQL Joins

## Definition
Operations that combine rows from two or more tables based on a related column.

## Why it matters
Joins are the core of relational querying. Choosing the right join and having supporting indexes determines correctness and performance.

## How it works
- **INNER JOIN**: rows matching in both tables.
- **LEFT (OUTER) JOIN**: all left rows; unmatched right columns are NULL.
- **RIGHT JOIN**: mirror of LEFT.
- **FULL OUTER JOIN**: all rows from both, NULLs where unmatched.
- **CROSS JOIN**: cartesian product (rarely intended).

```sql
SELECT o.id, c.name
FROM orders o
JOIN customers c ON c.id = o.customer_id      -- INNER
LEFT JOIN refunds r ON r.order_id = o.id      -- keep orders without refunds
WHERE o.status = 'SETTLED';
```

## Production usage
Index the join keys ([[Indexes]]). Watch for accidental row multiplication when joining one-to-many collections (use aggregation or fetch one collection at a time — see [[Hibernate-N-Plus-One]]). Verify plans with [[EXPLAIN]].

## Trade-offs
- Joins in the DB are usually far cheaper than N round-trips from the app.

## Common mistakes
- Missing indexes on join columns → nested-loop scans.
- Cartesian explosions from unintended CROSS JOINs or multi-collection joins.

## Interview questions
- INNER vs LEFT JOIN?
- Why can a join return more rows than expected?

## Related concepts
- [[Indexes]]
- [[EXPLAIN]]
- [[Query-Optimization]]
