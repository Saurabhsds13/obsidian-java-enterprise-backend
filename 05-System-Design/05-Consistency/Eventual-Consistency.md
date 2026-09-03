---
type: concept
domain: system-design
topic: consistency
difficulty: medium
status: inbox
tags: [system-design]
---

# Eventual Consistency

## Definition
A consistency model where, given no new updates, all replicas eventually converge to the same value — reads may be stale in the meantime.

## Why it matters
It's the pragmatic consistency model behind highly available, scalable systems (AP under [[CAP-Theorem]]), replicated databases, and caches.

## How it works
- Writes propagate asynchronously; replicas may briefly disagree ([[Replication]] lag).
- Conflicts are resolved by rules (last-write-wins, version vectors, CRDTs).
- Patterns to make it usable: **read-your-own-writes** (route recent writers to the primary), idempotent updates ([[Idempotency]]), and reconciliation jobs.

## Production usage
Fine for feeds, counts, product catalogs, and caches. Avoid for invariants that must be exact at read time (e.g. account balance at point of charge → prefer strong consistency there).

## Trade-offs
- Availability and low latency vs temporary staleness and conflict-resolution complexity.

## Common mistakes
- Using it where strict correctness is required.
- Ignoring read-your-own-writes UX issues.

## Interview questions
- When is eventual consistency acceptable?
- How do you handle read-your-own-writes?

## Related concepts
- [[CAP-Theorem]]
- [[Replication]]
- [[Kafka]]
