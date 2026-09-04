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
The mechanism by which services locate the current network addresses of their dependencies in an environment where instances are ephemeral (autoscaling, rolling deploys, failures). Two models: **client-side** (client queries a registry and load-balances itself) and **server-side** (a load balancer/gateway/platform resolves the target).

## Why it matters
Hardcoded endpoints break the moment instances scale or move. Discovery + health is what keeps routing correct in dynamic/containerized systems, and it's a standard piece of any microservices design answer.

## How it works — the mechanism
- **Registry** (Consul, Eureka, etcd, or the platform's DNS): instances **register** on startup and send **heartbeats**; the registry evicts instances that stop heartbeating (or whose health check fails).
- **Client-side discovery**: client asks the registry for healthy instances, caches them, and picks one (its own load balancing). Fewer hops, but discovery logic lives in every client/language.
- **Server-side discovery**: client calls a stable virtual endpoint; a load balancer resolves it. Simpler clients, one extra hop.
- **Kubernetes**: mostly solved by the platform — a `Service` gives a stable DNS name + virtual IP in front of changing pods; kube-proxy/CoreDNS handle resolution and health via readiness probes.

## Enterprise example — Kubernetes Service abstraction
```text
orders-svc (Service, stable DNS: orders-svc.default.svc.cluster.local)
   ├── pod A (ready)     <- readiness probe gates traffic
   ├── pod B (ready)
   └── pod C (NotReady)  <- excluded until healthy
Callers just use http://orders-svc/... ; pods can churn freely.
```

## Trade-offs
| | Client-side | Server-side |
|--|-------------|-------------|
| Hops | fewer (direct) | +1 (via LB) |
| Client complexity | high (per language) | low |
| Control point | distributed | centralized (easy policy/observability) |
| Example | Eureka + Ribbon | k8s Service, API gateway |

## Common mistakes (senior-level)
- Stale registry entries from missing health checks / no deregistration on shutdown → traffic to dead nodes.
- Aggressive client-side caching of instances without TTL/refresh → routing to removed pods.
- Reinventing discovery when the platform (k8s) already provides it.
- No readiness gating → traffic hits pods before they're warmed up (see [[Actuator]] probes).

## Interview questions (staff+)
- Client-side vs server-side discovery — trade-offs.
- How does Kubernetes handle service discovery (Service, DNS, readiness)?
- How do you avoid routing to a just-terminated instance?
- Why are heartbeats/health checks essential to a registry?

## Related concepts
- [[Load-Balancing]]
- [[Scalability]]
- [[Actuator]]
