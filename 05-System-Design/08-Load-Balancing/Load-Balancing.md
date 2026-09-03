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
Distributing incoming traffic across multiple servers so no single node is overwhelmed, enabling horizontal [[Scalability|scaling]] and availability.

## Why it matters
It's what makes scale-out and zero-downtime deploys possible, and it removes single points of failure at the app tier.

## How it works
- **Layer 4** (transport): routes by IP/port, fast, protocol-agnostic.
- **Layer 7** (application): routes by HTTP path/host/headers, enables smart routing and TLS termination.
- Algorithms: round-robin, least-connections, weighted, IP-hash (sticky).
- **Health checks** remove unhealthy nodes from rotation.

## Production usage
Prefer **stateless** services so any node can handle any request (avoid sticky sessions; store session state in [[Redis]]). Combine with autoscaling and graceful draining on deploy.

## Trade-offs
- L7 is smarter but heavier than L4. Sticky sessions simplify state but hurt balancing and failover.

## Common mistakes
- Relying on sticky sessions instead of externalizing state.
- No health checks → traffic sent to dead nodes.

## Interview questions
- L4 vs L7 load balancing?
- How do health checks and draining work?

## Related concepts
- [[Scalability]]
- [[Caching-Strategies]]
- [[Service-Discovery]]
