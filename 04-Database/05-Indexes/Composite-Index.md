---
type: concept
domain: database
topic: indexes
difficulty: hard
status: inbox
tags: [database]
---

# Composite Index

## Definition
A B+tree index on multiple columns in a defined order. The keys are sorted lexicographically by the column order, which is exactly why order determines the queries it can serve.

## Why it matters
A single well-ordered composite index can serve filtering, joining, sorting, and covering in one structure — or be completely useless if the columns are ordered wrong. It's a favorite interview probe because the **leftmost prefix rule** is unintuitive until you picture the sort order.

## How it works — the mechanism

### Leftmost prefix rule
An index on `(a, b, c)` is sorted by `a`, then `b` within equal `a`, then `c`. So it supports:
- `WHERE a = ?`
- `WHERE a = ? AND b = ?`
- `WHERE a = ? AND b = ? AND c = ?`
- `WHERE a = ? AND b > ?` (range on the last used column)

It does **not** support `WHERE b = ?` alone or `WHERE c = ?` — there's no contiguous run to seek. (`WHERE a = ? AND c = ?` uses only the `a` part, then filters `c`.)

### The equality-then-range ordering rule
Put **equality** predicates first, then **one range**, then columns for **sort**. A range column "stops" further index usage for seeking — anything after a range in the key can't be used for the seek, only as a filter. So `(status, created_at)` serves `status = ? AND created_at > ?` well; `(created_at, status)` does not.

### Sort elimination
If the query's `ORDER BY` matches the index order (including direction), the engine returns rows already sorted — no separate sort step. Mixed directions need matching mixed-direction index columns.

## Enterprise example
```sql
-- Query: WHERE tenant_id = ? AND status = ? ORDER BY created_at DESC LIMIT 20
-- Right order: equalities (tenant_id, status) then sort column (created_at DESC)
CREATE INDEX idx_orders_tenant_status_created
  ON orders (tenant_id, status, created_at DESC);
-- This one seeks to the exact (tenant,status) slice and reads it already sorted -> no filesort.
```

## Trade-offs
- One composite index often replaces several single-column indexes (fewer structures to maintain).
- More columns → larger index, more write cost; include only what queries use.
- Column order is query-specific — optimizing for one query shape can leave another unindexed.

## Common mistakes (senior-level)
- Expecting `(a, b)` to help a query filtering only on `b`.
- Putting a range column before an equality column → kills seek on later columns.
- Ignoring `ORDER BY` direction (index `ASC` can't cheaply serve `DESC` unless the engine reverse-scans, which it can, but mixed directions can't).
- Creating both `(a)` and `(a, b)` — the former is redundant (the composite already covers the `a` prefix).

## Interview questions (staff+)
- Explain the leftmost prefix rule with the sort-order intuition.
- Why put equality columns before the range column?
- How does a composite index eliminate a sort step?
- Given `WHERE a=? AND b>? ORDER BY c`, what's the ideal column order and why?
- When is `(a)` redundant given `(a, b)`?

## Related concepts
- [[Indexes]]
- [[EXPLAIN]]
- [[Query-Optimization]]
