---
type: concept
domain: backend
topic: observability
difficulty: medium
status: inbox
tags: [backend, production]
---

# Observability

## Definition
The ability to understand a system's internal state from its outputs: **logs**, **metrics**, and **traces** (the "three pillars"), tied together by correlation.

## Why it matters
You can't debug or operate what you can't see. Observability turns production incidents from guesswork into investigation.

## How it works
- **Logs**: structured (JSON) events with a **correlation/trace ID** so a request can be followed across services.
- **Metrics**: numeric time series (latency percentiles, error rate, throughput, saturation) — via Micrometer → Prometheus.
- **Traces**: spans across services showing where time goes (OpenTelemetry).
- Define **SLIs** (measured indicators) and **SLOs** (targets), and alert on SLO burn.

```text
request -> gateway [traceId=abc] -> order-svc [abc] -> payment-svc [abc]
logs, metrics, and spans all tagged abc -> one story
```

## Production usage
Emit RED metrics (Rate, Errors, Duration) per endpoint; propagate correlation IDs; alert on symptoms (SLOs) not causes. Back it with [[Actuator]] + Prometheus + a tracing backend.

## Trade-offs
- More telemetry = more insight but more cost/cardinality; sample traces and bound label cardinality.

## Common mistakes
- Unstructured logs with no correlation ID.
- Alerting on CPU instead of user-facing SLOs.

## Interview questions
- The three pillars and how they connect?
- SLI vs SLO vs SLA?

## Related concepts
- [[Actuator]]
- [[Circuit-Breaker]]
