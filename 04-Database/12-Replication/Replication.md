---
type: concept
domain: database
topic: replication
difficulty: medium
status: inbox
tags: [database]
---

# Replication

## Definition
Maintaining copies of a database on multiple nodes, typically a primary (writes) with one or more replicas (reads).

## Why it matters
Replication scales reads, improves availability, and enables failover — foundational for growing databases.

## How it works
- **Primary–replica**: writes go to the primary; changes stream to replicas.
- **Synchronous**: primary waits for replica ack (stronger durability, higher latency).
- **Asynchronous**: primary doesn't wait (lower latency, **replication lag** → replicas serve slightly stale reads).
- **Failover**: promote a replica if the primary dies.

## Production usage
Route reads to replicas, writes to the primary. Design for lag: read-your-own-writes may need to hit the primary right after a write. Combine with [[Connection-Pooling]] per endpoint.

## Trade-offs
- Async replication trades consistency ([[Eventual-Consistency]]) for latency/availability. More replicas = more read capacity, more operational overhead.

## Common mistakes
- Reading from a replica immediately after writing and seeing stale data.
- Assuming replicas add write capacity (they don't — see [[Sharding]]).

## Interview questions
- Sync vs async replication trade-offs?
- How do you handle replication lag?

## Related concepts
- [[Sharding]]
- [[Eventual-Consistency]]
- [[Connection-Pooling]]
