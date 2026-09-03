---
type: concept
domain: spring
topic: spring-boot
difficulty: easy
status: inbox
tags: [spring]
---

# Actuator

## Definition
Spring Boot Actuator exposes production-ready endpoints for health, metrics, info, and diagnostics over HTTP or JMX.

## Why it matters
It's the built-in [[Observability]] surface: health checks for orchestrators, metrics for dashboards, and runtime insight.

## How it works
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      probes:
        enabled: true   # /actuator/health/liveness and /readiness
```
- `/actuator/health` aggregates health indicators (DB, disk, custom).
- Micrometer backs `/actuator/metrics` and `/actuator/prometheus`.

## Production usage
Wire liveness/readiness probes to Kubernetes. Scrape `/actuator/prometheus`. Restrict exposure and secure endpoints — never expose `env`/`heapdump` publicly.

## Trade-offs
- Exposing too many endpoints leaks internals; expose only what you monitor.

## Common mistakes
- Exposing all endpoints (`include: "*"`) in production.
- Using the same probe for liveness and readiness.

## Interview questions
- Difference between liveness and readiness?
- How do you add a custom health indicator?

## Related concepts
- [[Observability]]
- [[Spring-Boot-Auto-Configuration]]
