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
Operations combining rows from multiple tables on a related predicate. Logically there are join *types* (INNER, LEFT/RIGHT/FULL OUTER, CROSS); physically the planner implements them with *algorithms* (nested loop, hash, merge) chosen by cost.

## Why it matters
Joins are the core of relational querying, and the gap between a fast and a catastrophic query is usually the join: missing indexes on join keys, or accidental row multiplication. Knowing both the logical semantics and the physical algorithms is the senior bar.

## How it works — the mechanism

### Join types
```sql
SELECT o.id, c.name, r.amount
FROM orders o
JOIN customers c   ON c.id = o.customer_id       -- INNER: only matches
LEFT JOIN refunds r ON r.order_id = o.id          -- keep all orders; refund cols NULL if none
-- RIGHT JOIN: mirror of LEFT; FULL OUTER: all rows both sides; CROSS: cartesian product
WHERE o.status = 'SETTLED';
```

### Physical algorithms (what the planner picks — see [[EXPLAIN]])
| Algorithm | How | Best when |
|-----------|-----|-----------|
| **Nested Loop** | for each outer row, probe inner | small outer + indexed inner |
| **Hash Join** | build hash of one side, probe other | large equijoins, inner not indexed; equality only |
| **Merge Join** | sort both, merge | inputs already sorted (index order); supports range |

### The row-multiplication trap
Joining a one-to-many relationship multiplies rows: 1 order × 3 items = 3 rows. Aggregating (`SUM`, `COUNT`) *after* such a join **double-counts**. Fix: aggregate in a subquery/CTE first, or join one collection at a time (this is the SQL cousin of the [[Hibernate-N-Plus-One|N+1 / cartesian]] problem).

## Enterprise example — avoid double-counting
```sql
-- WRONG: joining items inflates the order total sum
-- RIGHT: pre-aggregate, then join
SELECT o.id, o.total, i.item_count
FROM orders o
JOIN (SELECT order_id, COUNT(*) AS item_count FROM order_items GROUP BY order_id) i
  ON i.order_id = o.id;
```

## Trade-offs
- A join executed in the DB is almost always far cheaper than N application round-trips ([[Hibernate-N-Plus-One]]) — push set logic into SQL.
- But a poorly-indexed or cartesian join can be worse than either — index the join keys and watch cardinality.

## Common mistakes (senior-level)
- Missing index on the join key → nested loop over a full inner scan (or a forced hash/merge with big memory).
- Aggregating over a one-to-many join without pre-aggregating → inflated sums/counts.
- Accidental CROSS JOIN (missing ON) or multi-collection joins → cartesian explosion.
- Filtering an outer-joined table in `WHERE` (turns a LEFT JOIN back into an INNER) instead of in the `ON` clause.

## Interview questions (staff+)
- INNER vs LEFT JOIN; how does putting the filter in `WHERE` vs `ON` change a LEFT JOIN?
- Nested loop vs hash vs merge — when does the planner choose each?
- Why can a join return more rows than expected, and how do you aggregate safely?
- Why is a DB join usually better than fetching + joining in the app?

## Related concepts
- [[Indexes]]
- [[EXPLAIN]]
- [[Query-Optimization]]
- [[Hibernate-N-Plus-One]]
