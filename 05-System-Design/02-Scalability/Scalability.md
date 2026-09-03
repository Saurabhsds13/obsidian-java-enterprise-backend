---
type: concept
domain: system-design
topic: scalability
difficulty: medium
status: inbox
tags: [system-design]
---

# Scalability

## Definition
The ability of a system to handle increased load by adding resources, ideally with near-linear cost.

## Why it matters
Scalability determines whether a system survives growth. It shapes nearly every system-design decision.

## How it works
- **Vertical scaling** (scale up): bigger machine. Simple, but has a ceiling and a single point of failure.
- **Horizontal scaling** (scale out): more machines behind a [[Load-Balancing|load balancer]]. Needs **stateless** services so any node can serve any request.
- Move state out of app nodes: to databases, caches ([[Redis]]), and object stores. Scale the data tier with [[Replication]] (reads) and [[Sharding]] (writes).
- Use async/[[Message-Queues|queues]] to absorb spikes.

## Production usage
Design services stateless; push sessions to a store; cache hot reads; queue heavy work. Identify the bottleneck (usually the database) before scaling blindly.

## Trade-offs
- Horizontal scaling adds coordination and consistency complexity; vertical is simpler but limited.

## Common mistakes
- Stateful app nodes preventing scale-out.
- Scaling the app tier while the DB is the bottleneck.

## Interview questions
- Horizontal vs vertical scaling?
- What has to be true for a service to scale horizontally?

## Related concepts
- [[Load-Balancing]]
- [[Caching-Strategies]]
- [[Replication]]
- [[Sharding]]
