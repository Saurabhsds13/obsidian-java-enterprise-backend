---
type: concept
domain: system-design
topic: fundamentals
difficulty: hard
status: inbox
tags: [system-design]
---

# CAP Theorem

## Definition
In a distributed data store, during a **network partition** (P) you must choose between **Consistency** (C — every read sees the latest acknowledged write, i.e. linearizability) and **Availability** (A — every request to a non-failed node returns a non-error response). You cannot have both *while partitioned*. When there is no partition, you can have both.

## Why it matters
CAP is the framing device for every replication/consistency decision. The nuance that separates seniors from juniors: partitions are **not optional** (real networks drop/delay packets), so the practical choice is **CP vs AP**, and the *steady-state* trade-off is captured better by **PACELC**.

## How it works — the mechanism
- The three properties: **C** = linearizable reads; **A** = every live node answers; **P** = system keeps operating despite dropped inter-node messages.
- Because P is mandatory, under a partition you pick:
  - **CP**: refuse/stall requests that can't guarantee the latest value (e.g. a system that requires a **quorum** — see below). Sacrifices availability to stay correct.
  - **AP**: keep answering from whatever node you can reach, accept divergence, reconcile later ([[Eventual-Consistency]]).

### Quorums (how CP is actually implemented)
With N replicas, read quorum **R** and write quorum **W**: if **W + R > N**, a read set and write set always overlap → you read the latest write (strong consistency). Common: N=3, W=2, R=2. Lowering W/R raises availability/latency but risks stale reads (AP-leaning). This is the dial behind Dynamo-style stores and Kafka's `acks`/ISR.

### PACELC (the better model)
**if Partition → A vs C, Else → Latency vs Consistency.** Even with no partition, synchronous replication for strong consistency costs latency; async replication cuts latency but allows staleness. Example classifications: a strongly-consistent SQL primary ≈ **PC/EC**; a Dynamo-style store ≈ **PA/EL**.

## Enterprise example — matching the choice to the domain
| System | Lean | Why |
|--------|------|-----|
| Payments / ledger | **CP** | double-spend/lost-money is unacceptable; reject under partition |
| Shopping cart, feed, presence | **AP** | availability > perfect freshness; reconcile later |
| Leader election / config (ZooKeeper/etcd) | **CP** | correctness of the single source of truth |
| DNS, CDN | **AP** | must always answer; staleness tolerable |

## Trade-offs
- **CP**: correctness during partitions, at the cost of rejected/slow requests (reduced availability).
- **AP**: always-on and low-latency, at the cost of temporary inconsistency and conflict-resolution complexity.
- The choice is often **per-operation**, not per-system (a bank may be CP for transfers, AP for showing marketing balances).

## Common mistakes (senior-level)
- Treating CAP as "pick any 2 of 3" — P isn't optional in practice.
- Claiming a system is "CA" (only possible in a single node / no network).
- Ignoring the *no-partition* latency-vs-consistency trade-off (that's why PACELC exists).
- Applying one global choice instead of per-operation consistency.

## Interview questions (staff+)
- State CAP precisely, then explain why the real choice is CP vs AP.
- How do quorums (W + R > N) give strong consistency, and what's the availability cost?
- Explain PACELC and classify a system you know.
- Would you make a payment system CP or AP? A shopping cart? Justify per operation.

## Related concepts
- [[Eventual-Consistency]]
- [[Replication]]
- [[Scalability]]
- [[Distributed-Locks]]
