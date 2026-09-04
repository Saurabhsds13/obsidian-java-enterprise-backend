---
type: concept
domain: system-design
topic: load-balancing
difficulty: medium
status: inbox
tags: [system-design]
---

# Load Balancing

## Definition
Distributing incoming traffic across a pool of backends so no node is overwhelmed, enabling horizontal [[Scalability|scale]], high availability, and zero-downtime deploys. Operates at **Layer 4** (transport: IP/port) or **Layer 7** (application: HTTP path/host/headers).

## Why it matters
It's the fabric that makes stateless services scalable and resilient. Senior depth: L4 vs L7 trade-offs, algorithm choice, health checking, connection draining, and why sticky sessions are usually the wrong answer.

## How it works — the mechanism
| | Layer 4 | Layer 7 |
|--|---------|---------|
| Routes on | IP/port (TCP/UDP) | HTTP path, host, headers, cookies |
| Sees payload | no | yes (can terminate TLS) |
| Speed | very fast | slightly heavier |
| Features | pass-through | path routing, retries, header rewrite, WAF |

### Algorithms
- **Round-robin** / **weighted** — even or capacity-proportional.
- **Least-connections** — send to the least-busy backend (good for uneven request costs).
- **Consistent hashing** — map key → backend so the same client/key hits the same node with minimal reshuffling when the pool changes (used for sticky caches, shard routing) — see below.
- **Latency/EWMA** — pick the fastest-responding backend.

### Health checks + draining
Active checks (`GET /actuator/health/readiness`) remove unhealthy nodes; on deploy, **connection draining** stops new traffic while letting in-flight requests finish → zero-downtime rollout.

### Consistent hashing (why it matters)
Plain `hash(key) % N` remaps almost everything when N changes. Consistent hashing places nodes on a ring; adding/removing a node only moves ~1/N of keys. **Virtual nodes** smooth out uneven distribution. This is the backbone of sharded caches and distributed stores.

## Enterprise example — stateless behind an L7 LB
```text
Client -> L7 LB (TLS termination, path routing, health checks)
            ├── /api/orders/*   -> orders-svc (autoscaled, stateless)
            └── /api/payments/* -> payments-svc
Session state -> Redis (NOT in-memory), so any node serves any request
```

## Trade-offs
- L7 is smarter (routing, retries, observability) but does more work per request; L4 is faster and protocol-agnostic.
- **Sticky sessions** simplify in-memory state but break even balancing and lose the session on node failure — prefer externalizing state to [[Redis]].

## Common mistakes (senior-level)
- In-memory sessions forcing sticky routing → poor balancing + failover data loss.
- No health checks / no draining → traffic to dead or deploying nodes.
- `hash % N` for sharding instead of consistent hashing → mass remap on scaling.
- Ignoring the LB as a single point of failure (need redundant LBs / DNS failover).

## Interview questions (staff+)
- L4 vs L7 — when do you need L7?
- Explain consistent hashing and virtual nodes; why not `hash % N`?
- How do health checks + connection draining enable zero-downtime deploys?
- Why avoid sticky sessions, and what's the alternative?

## Related concepts
- [[Scalability]]
- [[Caching-Strategies]]
- [[Service-Discovery]]
