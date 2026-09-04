---
type: concept
domain: system-design
topic: consistency
difficulty: hard
status: inbox
tags: [system-design]
---

# Eventual Consistency

## Definition
A consistency model guaranteeing that, absent new writes, all replicas **converge** to the same value eventually — reads may return stale data in the interim. It is the practical model behind AP systems ([[CAP-Theorem]]), replicated stores, caches, and event-driven pipelines.

## Why it matters
Most scalable systems are eventually consistent somewhere. The senior skill is knowing *where* it's acceptable, how to make it usable for end users (session guarantees), and how conflicts are resolved.

## How it works — the mechanism
- Writes propagate asynchronously ([[Replication]] lag, async [[Message-Queues]], cache TTLs), so replicas temporarily disagree.
- **Convergence + conflict resolution**:
  - **Last-Write-Wins (LWW)** by timestamp — simple, but clock skew can drop writes.
  - **Version vectors / vector clocks** — detect concurrent writes, surface conflicts.
  - **CRDTs** (Conflict-free Replicated Data Types) — data types (counters, sets) that merge deterministically without coordination.
- **Session (client-centric) guarantees** make it tolerable:
  - **Read-your-writes**: a user sees their own updates (route recent writers to the primary or a sticky replica).
  - **Monotonic reads**: never see time go backwards (pin a user to one replica).
- **Idempotency** ([[Idempotency]]) makes re-delivered/retried updates safe.

## The consistency spectrum (know where EC sits)
```
Strong (linearizable) > Sequential > Causal > Read-your-writes/Monotonic > Eventual
   more coordination/latency  <---------------------------------->  more availability
```

## Enterprise example — where EC is fine vs not
| Data | Model | Rationale |
|------|-------|-----------|
| Product catalog, feed, view counts | eventual | availability + scale; slight staleness invisible |
| Cart (single user) | read-your-writes | user must see their own adds |
| Account balance *at point of charge* | strong | correctness required → CP path |
| Analytics / search index | eventual | rebuilt async from events |

## Trade-offs
- Advantages: high availability, low latency, horizontal scale, partition tolerance.
- Disadvantages: stale reads, conflict-resolution complexity, harder reasoning/testing, "read-your-writes" UX bugs if ignored.

## Common mistakes (senior-level)
- Using EC where an invariant must hold at read time (money, inventory decrement).
- Ignoring read-your-own-writes → user updates something and it "disappears" on refresh.
- LWW with unsynchronized clocks silently losing writes.
- Assuming a read replica is fresh immediately after writing to the primary.

## Interview questions (staff+)
- Place eventual consistency on the spectrum from linearizable to eventual.
- How do you give a user read-your-writes on an eventually consistent store?
- Compare LWW, vector clocks, and CRDTs for conflict resolution.
- Give one place EC is fine and one where it's dangerous, and why.

## Related concepts
- [[CAP-Theorem]]
- [[Replication]]
- [[Kafka]]
- [[Idempotency]]
