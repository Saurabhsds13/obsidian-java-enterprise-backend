---
type: interview
domain: database
topic: query-optimization
difficulty: hard
status: inbox
tags: [interview, production]
---

# How would you handle a database that becomes the bottleneck?

## Question
> Your database is saturated and latency is climbing. How do you diagnose and fix it?

## Short answer
Measure first (slow queries, plans, pool health), then optimize queries/indexes, cache hot reads, and only then scale with replicas or sharding.

## Detailed answer
A layered approach, cheapest first:
1. **Diagnose**: slow-query log, [[EXPLAIN]] plans, [[Connection-Pooling|pool]] metrics, lock/deadlock stats.
2. **Query/index**: fix N+1 ([[Hibernate-N-Plus-One]]), add/adjust [[Indexes]], rewrite bad queries ([[Query-Optimization]]).
3. **Reduce load**: [[Caching]] hot reads, keyset [[Pagination]], batch writes.
4. **Scale reads**: [[Replication]] read replicas (mind lag).
5. **Scale writes**: [[Sharding]] as a last resort.
6. **Right-size** the connection pool to the DB.

## Example
Slow endpoint → EXPLAIN shows a seq scan → add composite index → p99 drops; add cache for the hottest read.

## Production relevance
The database is the most common backend bottleneck; disciplined, measured tuning beats blind scaling.

## Common mistake
Adding app instances or a bigger pool while the DB is the constraint (makes it worse).

## Follow-up questions
- How do you handle replication lag for read-your-own-writes?
- When is sharding justified?

## Related concepts
- [[Query-Optimization]]
- [[Indexes]]
- [[Replication]]
- [[Connection-Pooling]]
