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
An index on multiple columns in a specific order. The column order determines which queries it can serve.

## Why it matters
The wrong column order makes a composite index useless for a query; the right order can serve filtering, sorting, and covering in one structure.

## How it works
- Follows the **leftmost prefix** rule: an index on `(a, b, c)` supports predicates on `a`, `a,b`, and `a,b,c` — but not on `b` alone.
- Put the most selective / equality-filtered column first, then range/sort columns.

```sql
-- Serves WHERE customer_id = ? ORDER BY created_at DESC
CREATE INDEX idx_orders_cust_created ON orders (customer_id, created_at DESC);
```

## Production usage
Design composite indexes around real query shapes. A single well-ordered composite index often replaces several single-column ones.

## Trade-offs
- More columns = larger index and more write overhead; only include what queries use.

## Common mistakes
- Expecting `(a, b)` to help a query filtering only on `b`.
- Ignoring sort direction when the query has `ORDER BY`.

## Interview questions
- Explain the leftmost prefix rule.
- How do you order columns in a composite index?

## Related concepts
- [[Indexes]]
- [[EXPLAIN]]
- [[Query-Optimization]]
