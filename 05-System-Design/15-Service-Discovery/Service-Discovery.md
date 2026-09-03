---
type: concept
domain: system-design
topic: service-discovery
difficulty: medium
status: inbox
tags: [system-design]
---

# Service Discovery

## Definition
The mechanism by which services find the network locations (host/port) of other services in a dynamic environment where instances come and go.

## Why it matters
In autoscaled/containerized systems, instance addresses change constantly. Hardcoding endpoints breaks; discovery keeps routing correct.

## How it works
- **Client-side discovery**: the client queries a registry (e.g. Consul, Eureka) and load-balances itself.
- **Server-side discovery**: a [[Load-Balancing|load balancer]]/gateway resolves the target (common in Kubernetes via its DNS + Services).
- Instances **register** on startup and send **heartbeats**; unhealthy ones are removed.

## Production usage
On Kubernetes, a `Service` gives a stable virtual IP/DNS name in front of changing pods — discovery is mostly handled by the platform. Elsewhere, a registry plus health checks does the job.

## Trade-offs
- Client-side is flexible but pushes logic into clients; server-side centralizes it but adds a hop.

## Common mistakes
- Stale registry entries from missing health checks/deregistration.

## Interview questions
- Client-side vs server-side discovery?
- How does Kubernetes handle it?

## Related concepts
- [[Load-Balancing]]
- [[Scalability]]
