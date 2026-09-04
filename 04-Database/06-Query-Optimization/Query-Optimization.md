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
The disciplined, evidence-driven practice of making queries faster by improving indexes, query shape, schema, and statistics — guided by execution plans ([[EXPLAIN]]) and prioritized by total impact (frequency × latency), not just the single slowest query.

## Why it matters
The DB is the most common backend bottleneck, and query tuning usually beats hardware. Architect-level means a repeatable method and knowing which lever fits which plan symptom.

## How it works — the method
1. **Find offenders**: `pg_stat_statements` / slow-query log ranked by **total time** (a 5 ms query run 10k×/s outweighs a 2 s report run hourly).
2. **Read the plan** ([[EXPLAIN]] ANALYZE, BUFFERS): locate the costly node; check estimate-vs-actual (stale stats?).
3. **Index** ([[Indexes]], [[Composite-Index]]): add/adjust for the predicate + sort shape; aim for index-only scans.
4. **Rewrite**: avoid `SELECT *`, functions on indexed columns, needless `DISTINCT`, correlated subqueries (often → joins); pick the right [[SQL-Joins|join]].
5. **Reduce work**: keyset [[Pagination]] instead of deep `OFFSET`; batch to kill [[Hibernate-N-Plus-One|N+1]]; project only needed columns.
6. **Maintain**: keep statistics fresh (`ANALYZE`/autovacuum); watch bloat and [[Connection-Pooling|pool]] health.

## Plan symptom → fix (cheat sheet)
| Symptom in plan | Likely fix |
|-----------------|-----------|
| Seq Scan on large table, selective predicate | add an index |
| Index Scan then many heap fetches | make it covering (index-only) |
| Sort node for `ORDER BY` | index in sort order |
| estimate ≪ actual rows | refresh statistics (`ANALYZE`) |
| Nested loop with huge inner | ensure inner is indexed, or force hash join via better stats |
| Deep `OFFSET` slow | keyset pagination |

## Enterprise example — deep offset → keyset
```sql
-- Slow: OFFSET 100000 scans and discards 100k rows
SELECT * FROM orders ORDER BY id LIMIT 50 OFFSET 100000;
-- Fast: seek by the last-seen key (uses the index, constant cost at any depth)
SELECT * FROM orders WHERE id > :lastId ORDER BY id LIMIT 50;
```

## Trade-offs
- Indexes speed reads but slow writes/consume storage.
- Denormalization speeds reads but risks consistency and complicates writes.
- Materialized views/precomputation cut read cost but add refresh/staleness.

## Common mistakes (senior-level)
- Tuning without reading the plan.
- Optimizing the *slowest* query instead of the *highest-total-time* one.
- `SELECT *` on wide tables (defeats covering indexes, ships useless bytes).
- Deep `OFFSET` pagination; N+1 from the ORM.
- Adding indexes without checking they're actually used (`EXPLAIN`), then paying write cost for nothing.

## Interview questions (staff+)
- Walk your end-to-end method for a slow endpoint.
- Why prioritize by total time, not worst single query?
- Give three query rewrites that unlock an index.
- Why is deep OFFSET slow and what replaces it?
- When is denormalization or a materialized view the right call?

## Related concepts
- [[EXPLAIN]]
- [[Indexes]]
- [[Composite-Index]]
- [[Connection-Pooling]]
- [[Hibernate-N-Plus-One]]
