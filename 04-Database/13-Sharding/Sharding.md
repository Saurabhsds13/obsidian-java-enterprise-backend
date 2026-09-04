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
Horizontally partitioning data across multiple independent databases (**shards**), each holding a disjoint subset, to scale **writes** and storage beyond a single machine. A **shard key** determines which shard a row lives on.

## Why it matters
[[Replication]] and caching scale reads; when a single primary can't absorb the **write** volume or the dataset outgrows one machine, sharding is the answer — but it's the highest-complexity lever, so it's a "last resort" decision an architect must justify.

## How it works — the mechanism

### Shard key + strategy
| Strategy | Mapping | Pros | Cons |
|----------|---------|------|------|
| **Range** | key ranges → shards | easy range scans | hotspots (e.g. time-ordered writes hit one shard) |
| **Hash** | `hash(key) → shard` | even distribution | no range scans; resharding is painful with `mod N` |
| **Consistent hashing** | key → ring position | minimal reshuffle when adding/removing shards | more moving parts |
| **Directory / lookup** | explicit map service | flexible, re-mappable | the lookup is a dependency/SPOF |

- **Consistent hashing + virtual nodes** avoids the mass remap that plain `hash % N` causes when shard count changes (see [[Load-Balancing]]).

### Choosing the shard key (the crux)
A good key gives **even distribution** *and* routes most queries to a **single shard**. E.g. shard by `tenant_id` for a multi-tenant SaaS so a tenant's queries stay on one shard. A bad key creates **hot shards** (celebrity user, monotonic timestamp) and forces cross-shard scatter-gather on common queries.

## The hard problems sharding introduces
- **Cross-shard queries**: joins/aggregations must scatter-gather and merge in the app → slow, complex.
- **Cross-shard transactions**: no single-DB ACID; need 2PC (slow, availability cost) or a **saga** ([[Message-Queues]]) with compensations.
- **Rebalancing**: adding shards moves data; consistent hashing minimizes but doesn't eliminate it.
- **Unique keys / global IDs**: auto-increment breaks across shards → use UUIDs or a Snowflake-style ID generator.

## Enterprise example — order it correctly
```text
Scale reads first:   cache (Redis) -> read replicas
Then partition:      table partitioning (single DB) for large tables
Only then shard:     by tenant_id (queries stay single-shard), consistent-hash routing
Global IDs:          Snowflake IDs so ids are unique across shards
```

## Trade-offs
- Scales writes/storage ~linearly, but sacrifices easy joins, cross-entity transactions, and operational simplicity.
- Shard late: exhaust caching + replicas + partitioning first; sharding is hard to undo.

## Common mistakes (senior-level)
- Sharding prematurely (before caching/replicas/partitioning).
- A shard key that causes hot shards or forces cross-shard queries for the common path.
- Auto-increment IDs across shards (collisions) instead of UUID/Snowflake.
- Expecting cross-shard ACID (need saga/2PC).
- `hash % N` routing → mass data movement when you add a shard (use consistent hashing).

## Interview questions (staff+)
- How do you choose a shard key, and what makes one bad?
- Range vs hash vs consistent-hash vs directory sharding.
- Why are cross-shard transactions hard, and what's the alternative?
- Why shard *after* replicas/caching, and why is it hard to reverse?
- How do you generate unique IDs across shards?

## Related concepts
- [[Replication]]
- [[Eventual-Consistency]]
- [[Scalability]]
- [[Load-Balancing]]
