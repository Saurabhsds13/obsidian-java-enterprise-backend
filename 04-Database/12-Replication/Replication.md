---
type: concept
domain: database
topic: replication
difficulty: hard
status: inbox
tags: [database]
---

# Replication

## Definition
Maintaining copies of a database across nodes — typically a **primary** (accepts writes) streaming changes to one or more **replicas** (serve reads / stand by for failover). It scales reads, improves availability, and enables disaster recovery.

## Why it matters
It's the first data-tier scaling lever after caching, and its central trade-off — **replication lag vs durability** — drives real correctness bugs (stale reads, read-your-writes) and failover decisions.

## How it works — the mechanism
- The primary ships its change log (Postgres WAL / MySQL binlog) to replicas, which replay it.
- **Synchronous**: primary waits for ≥1 replica to acknowledge before commit → no data loss on failover, but higher write latency and reduced availability if a replica is slow/down.
- **Asynchronous** (common default): primary commits without waiting → low latency, but a window of **replication lag** where replicas are stale and a crash can lose the un-shipped tail.
- **Semi-synchronous**: wait for one replica's *receipt* (not full apply) — a middle ground.
- **Failover**: promote a replica when the primary dies; needs fencing/STONITH to avoid **split-brain** (two primaries accepting writes).

## The lag problem (read-your-writes)
Route reads to replicas, writes to the primary — but a user who just wrote may read a stale replica and "lose" their change. Mitigations:
- Route a user's reads to the **primary for a short window** after they write.
- Track a **write LSN/timestamp** and only read from a replica caught up past it.
- Accept staleness where it's harmless ([[Eventual-Consistency]]).

## Enterprise example — read/write routing
```java
// Writes -> primary; reads -> replica, EXCEPT right after a write (read-your-writes)
@Transactional                     // primary
public void updateProfile(...) { ... }

@Transactional(readOnly = true)    // routed to a replica via a routing DataSource
public ProfileView getProfile(...) { ... }
```
(Spring: `AbstractRoutingDataSource` keyed off `@Transactional(readOnly=...)`.)

## Trade-offs
| | Synchronous | Asynchronous |
|--|-------------|--------------|
| Data loss on failover | none | possible (lag window) |
| Write latency | higher | low |
| Availability if replica slow | reduced | unaffected |
| Consistency of replica reads | strong-ish | eventual |

Replicas add **read** capacity and availability — they do **not** add write capacity (that's [[Sharding]]).

## Common mistakes (senior-level)
- Reading a replica immediately after writing → stale data (no read-your-writes handling).
- Assuming replicas scale writes (they don't).
- Async replication + failover losing the unreplicated tail without acknowledging the risk.
- Split-brain on failover without fencing.
- Ignoring replica lag in dashboards until it breaks a feature.

## Interview questions (staff+)
- Synchronous vs asynchronous vs semi-synchronous — trade-offs.
- How do you give read-your-writes when reads go to replicas?
- Why don't replicas scale writes?
- What is split-brain and how does failover avoid it?
- How do you monitor and bound replication lag?

## Related concepts
- [[Sharding]]
- [[Eventual-Consistency]]
- [[Connection-Pooling]]
- [[CAP-Theorem]]
