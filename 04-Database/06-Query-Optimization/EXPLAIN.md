---
type: concept
domain: database
topic: query-optimization
difficulty: hard
status: inbox
tags: [database]
---

# EXPLAIN

## Definition
A command that reveals the **query planner's execution plan**: the access methods (seq scan, index scan, index-only scan), join algorithms, order of operations, and cost/row estimates. `EXPLAIN ANALYZE` actually runs the query and reports **real** timings and row counts alongside the estimates.

## Why it matters
It's the evidence for "measure before optimizing." A staff-level engineer reads a plan to find the expensive node, spots estimate-vs-actual gaps (stale statistics), and knows which fix (index, rewrite, stats refresh) applies.

## How it works — reading a plan

### Scan node types (cheapest signal)
| Node | Meaning | Usually |
|------|---------|---------|
| **Index Only Scan** | answered from the index alone | best |
| **Index Scan** | seek index → fetch rows | good for selective predicates |
| **Bitmap Index Scan** | many matches → build bitmap → fetch | medium-selectivity |
| **Seq Scan** | full table read | fine for small tables / low selectivity; a red flag on large ones |

### Join algorithms
- **Nested Loop**: for each outer row, probe inner (great when inner is indexed and outer is small).
- **Hash Join**: build a hash of one side, probe with the other (great for large, unindexed equijoins).
- **Merge Join**: both inputs sorted, merged (great when already sorted / index-ordered).
The planner picks based on estimated cardinalities — wrong estimates → wrong join → slow query.

### The key diagnostic: estimated vs actual rows
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ... ;
--   Seq Scan on orders  (cost=... rows=1000  ...) (actual ... rows=2100000 ...)
--                                  ^^^^ estimate           ^^^^^^^ reality
```
A large **estimate ≠ actual** gap means **stale statistics** → the planner chose a bad plan. Fix with `ANALYZE` (refresh stats) before blaming the query. `BUFFERS` shows cache hits vs disk reads.

## Enterprise example — diagnosing a slow endpoint
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT status, total FROM orders
WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 50;
-- Bad:  Seq Scan + Sort (no index)   -> add composite index (customer_id, created_at DESC)
-- Good: Index Only Scan using idx... -> no table fetch, no sort
```

## Trade-offs
- `EXPLAIN` is free (plan only); `EXPLAIN ANALYZE` **executes** the query — dangerous for writes (wrap in a transaction and `ROLLBACK`).
- Plans are cost-model estimates; the model can be wrong (skew, correlation) — validate with `ANALYZE` actuals.

## Common mistakes (senior-level)
- Optimizing by guesswork instead of reading the plan.
- Ignoring the estimate-vs-actual gap (stale stats) and adding indexes that don't help.
- Running `EXPLAIN ANALYZE` on an `UPDATE/DELETE` in prod without a transaction/rollback.
- Assuming a Seq Scan is always bad (on a tiny table it's optimal).

## Interview questions (staff+)
- Walk through diagnosing a slow query with `EXPLAIN ANALYZE`.
- Nested loop vs hash vs merge join — when does the planner pick each?
- What does a large estimated-vs-actual row gap tell you, and how do you fix it?
- Index Scan vs Index Only Scan vs Bitmap Scan?
- Why is `EXPLAIN ANALYZE` risky on writes?

## Related concepts
- [[Indexes]]
- [[Composite-Index]]
- [[Query-Optimization]]
- [[SQL-Joins]]
