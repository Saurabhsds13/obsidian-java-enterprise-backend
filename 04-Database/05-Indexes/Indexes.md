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
Auxiliary data structures — almost always **B+trees** — that let the engine locate rows without scanning the whole table, turning O(n) scans into O(log n) seeks plus efficient range reads. Other kinds: hash (equality only), GIN/GiST (full-text, arrays, geo), bitmap (low-cardinality analytics).

## Why it matters
The database is the usual backend bottleneck, and indexing is the highest-leverage query-tuning lever. Architect depth means understanding the B+tree shape, selectivity, covering/index-only scans, clustered vs secondary indexes, and the write cost — not just "add an index."

## How it works — the mechanism

### Why B+tree (not a plain B-tree or hash)
- All values live in the **leaf level**; leaves are a **doubly-linked list** → excellent **range scans** and `ORDER BY` without sorting.
- Internal nodes hold only keys → high fan-out → shallow tree (billions of rows in ~3–4 levels → ~3–4 page reads per lookup).
- Hash indexes are O(1) for `=` but useless for ranges/sorting; B+tree serves `=`, `<`, `>`, `BETWEEN`, prefix `LIKE 'abc%'`, and `ORDER BY`.

### Clustered vs secondary (storage model matters)
- **Clustered index** (InnoDB primary key): the table *is* the B+tree, rows stored in PK order. Secondary indexes store the **PK** as the row pointer → a secondary lookup does a second traversal ("bookmark lookup") unless covered.
- **Heap + secondary** (PostgreSQL): indexes point to a physical tuple id (`ctid`); PG has no clustered index (hence the visibility-map/index-only-scan machinery).

### Selectivity, cardinality, covering
- **Selectivity** = fraction of rows a predicate returns. The planner uses statistics; a low-selectivity predicate (returns most rows) makes an index *worse* than a scan.
- **Covering / index-only scan**: if the index contains every column the query needs, the engine never touches the table. Add columns via composite indexes or `INCLUDE` (Postgres) / wide secondary (MySQL).

## Enterprise example
```sql
-- Covering index for a hot list query (filter + sort + returned columns):
CREATE INDEX idx_orders_cust_created
  ON orders (customer_id, created_at DESC) INCLUDE (status, total);   -- PG: index-only scan
-- Verify it's used and index-only:
EXPLAIN (ANALYZE, BUFFERS)
SELECT status, total FROM orders
WHERE customer_id = 42 ORDER BY created_at DESC LIMIT 50;
```

## Trade-offs
- Reads: dramatic speedup for selective predicates, ranges, sorts, joins.
- Writes: every index is maintained on `INSERT/UPDATE/DELETE` → write amplification + storage; over-indexing slows writes and bloats.
- Low-cardinality columns (boolean, status with 3 values) rarely benefit from a B+tree (consider partial/filtered indexes instead).

## Common mistakes (senior-level)
- Indexing every column ("just in case") → write penalty + planner confusion.
- Function/expression on the indexed column in `WHERE` (`WHERE lower(email)=...`) defeats the index → use an expression index.
- Leading wildcard `LIKE '%abc'` can't use a B+tree.
- Ignoring selectivity — an index on a mostly-true boolean is dead weight.
- Forgetting that a secondary-index lookup in InnoDB costs a second (PK) traversal unless covering.

## Interview questions (staff+)
- Why B+tree over a plain B-tree or hash index?
- Clustered vs secondary index — what does a secondary lookup actually do in InnoDB?
- What is a covering / index-only scan and how do you get one?
- How does selectivity change whether the planner uses an index?
- Why does `WHERE lower(col) = ?` skip the index, and how do you fix it?

## Related concepts
- [[Composite-Index]]
- [[EXPLAIN]]
- [[Query-Optimization]]
- [[SQL-Joins]]
