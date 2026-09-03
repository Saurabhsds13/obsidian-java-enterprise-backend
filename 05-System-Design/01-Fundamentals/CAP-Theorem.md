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
In a distributed system, during a **network Partition** you can guarantee either **Consistency** or **Availability**, not both. Without a partition, you can have both.

## Why it matters
CAP frames the fundamental trade-off in distributed data systems and guides database and design choices.

## How it works
- **C** (consistency): every read sees the latest write.
- **A** (availability): every request gets a (non-error) response.
- **P** (partition tolerance): the system keeps working despite dropped/delayed messages between nodes.
- Partitions are unavoidable in real networks, so the real choice under a partition is **CP** vs **AP**:
  - **CP**: reject/timeout to stay consistent (e.g. a system requiring quorum).
  - **AP**: keep serving, allow stale/divergent data, reconcile later ([[Eventual-Consistency]]).

## Beyond CAP: PACELC
PACELC extends it: **if Partition then A-vs-C, Else Latency-vs-Consistency** — even without partitions there's a latency/consistency trade-off.

## Production usage
Match the choice to the domain: payments lean CP for correctness; feeds/carts often lean AP for availability.

## Trade-offs
- CP: correctness, but reduced availability during partitions.
- AP: availability, but temporary inconsistency.

## Common mistakes
- Treating CAP as "pick 2 of 3" always (P is mandatory in practice).

## Interview questions
- Explain CAP and PACELC.
- Would you choose CP or AP for a payment system? Why?

## Related concepts
- [[Eventual-Consistency]]
- [[Replication]]
- [[Scalability]]
