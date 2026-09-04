---
type: concept
domain: system-design
topic: scalability
difficulty: hard
status: inbox
tags: [system-design]
---

# Scalability

## Definition
The ability of a system to sustain increased load (throughput, data, users) by adding resources, ideally with **near-linear** cost and no drop in latency SLOs. Vertical scaling grows one machine; horizontal scaling adds machines.

## Why it matters
Every system-design answer is ultimately about scaling a bottleneck. The senior skill is *identifying the actual bottleneck* (usually the data tier) and applying the cheapest effective lever, backed by capacity math — not reflexively "add more servers."

## How it works — the levers (cheapest first)
1. **Make services stateless** → any node handles any request → horizontal scale behind a [[Load-Balancing|load balancer]]. Push session/state to [[Redis]] or the DB.
2. **Cache** hot reads ([[Caching-Strategies]]) — often the biggest single win; offloads the DB.
3. **Scale reads** with [[Replication|read replicas]] (watch lag → [[Eventual-Consistency]]).
4. **Async + queues** ([[Message-Queues]]) to absorb spikes and decouple slow work.
5. **Scale writes** with [[Sharding]] (partition by key) — last resort, high complexity.
6. **Scale up** (bigger box) when simplest and within ceiling.

### Laws that bound scaling
- **Amdahl's Law**: speedup is capped by the serial fraction. If 5% of work is serial, max speedup ≈ 20× no matter how many cores/nodes.
- **Universal Scalability Law (USL)**: adds a *coherency/coordination* penalty — beyond a point, adding nodes **reduces** throughput due to cross-node coordination (locks, chatty consensus). Explains why "just add nodes" eventually backfires.
- **Little's Law** (`L = λ × W`): concurrent requests = arrival rate × latency — used to size pools/instances.

## Enterprise example — capacity sizing with Little's Law
```
Target: 5,000 req/s at 40 ms avg service time.
Concurrent in-flight = 5000 × 0.040 = 200 requests.
If one instance handles ~50 concurrent (thread/conn limited),
need ~200 / 50 = 4 instances + headroom (say 6) behind the LB.
```

## Trade-offs
| | Vertical (scale up) | Horizontal (scale out) |
|--|---------------------|------------------------|
| Complexity | low | high (statelessness, coordination) |
| Ceiling | hardware limit | ~unbounded |
| Failure | single point | resilient |
| Cost curve | steep at high end | more linear |
| Consistency | trivial | harder (partitions) |

## Common mistakes (senior-level)
- Scaling the **app tier** while the **database** is the bottleneck (makes it worse — more connections hammering the DB).
- Stateful app nodes (in-memory sessions) blocking horizontal scale.
- Sharding prematurely (before exhausting cache + replicas).
- Ignoring coordination cost (USL) — adding nodes past the coherency knee.
- No capacity math — guessing instance counts.

## Interview questions (staff+)
- Give the ordered levers to scale a read-heavy service; where does the DB fit?
- Size instances for 5k req/s at 40 ms (Little's Law).
- Amdahl vs USL — why can adding nodes reduce throughput?
- What must be true for a service to scale horizontally?
- Vertical vs horizontal trade-offs.

## Related concepts
- [[Load-Balancing]]
- [[Caching-Strategies]]
- [[Replication]]
- [[Sharding]]
