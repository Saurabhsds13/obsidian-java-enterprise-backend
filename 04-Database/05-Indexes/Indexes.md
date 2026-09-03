---
type: concept
domain: database
topic: indexes
difficulty: hard
status: inbox
tags: [database]
---

# Indexes

## Definition
Auxiliary data structures (usually B-trees) that let the database find rows without scanning the whole table.

## Why it matters
Indexes are the primary lever for query performance. The difference between an indexed lookup and a full scan can be milliseconds vs seconds.

## How it works
- A **B-tree** index keeps keys sorted, giving O(log n) lookups, range scans, and ordered reads.
- The **query planner** chooses an index based on **selectivity** (how many rows a predicate filters) and statistics.
- A **covering index** includes all columns a query needs, so the DB never touches the table ("index-only scan").

```sql
CREATE INDEX idx_orders_customer ON orders (customer_id);
-- range + order benefit:
CREATE INDEX idx_orders_created ON orders (created_at DESC);
```

## Production usage
Index columns used in `WHERE`, join keys, and `ORDER BY`. Verify with [[EXPLAIN]]. Consider [[Composite-Index|composite indexes]] for multi-column filters.

## Trade-offs
- Speeds reads but slows writes (each index maintained on insert/update/delete) and uses storage. Low-selectivity indexes (e.g. boolean) rarely help.

## Common mistakes
- Indexing everything.
- Wrong column order in composite indexes (see [[Composite-Index]]).
- Functions on indexed columns in `WHERE` defeating the index.

## Interview questions
- What is selectivity and why does it matter?
- What is a covering index?

## Related concepts
- [[Composite-Index]]
- [[EXPLAIN]]
- [[Query-Optimization]]
- [[SQL-Joins]]
