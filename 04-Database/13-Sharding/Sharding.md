---
type: concept
domain: database
topic: sharding
difficulty: hard
status: inbox
tags: [database]
---

# Sharding

## Definition
Horizontally partitioning data across multiple databases (shards), each holding a subset, to scale **writes** and storage beyond one machine.

## Why it matters
[[Replication]] scales reads but not writes. When a single primary can't keep up, sharding distributes the write load.

## How it works
- A **shard key** decides which shard a row lives on.
- Strategies: **range** (by key range), **hash** (hash of key → shard), **directory** (lookup table).
- **Consistent hashing** minimizes data movement when adding/removing shards.

```text
shard = hash(user_id) % N   -> user's data always on the same shard
```

## Production usage
Choose a shard key with even distribution and that matches query patterns (so most queries hit one shard). Shard late — it's a major complexity increase. Prefer scaling up + replicas + caching first.

## Trade-offs
- Scales writes/storage, but cross-shard queries, joins, and transactions become hard; rebalancing is painful; hot keys skew load.

## Common mistakes
- Poor shard key → hot shards and cross-shard queries everywhere.
- Sharding prematurely.

## Interview questions
- How do you pick a shard key?
- Why is cross-shard transaction hard?

## Related concepts
- [[Replication]]
- [[Eventual-Consistency]]
- [[Scalability]]
