---
type: concept
domain: backend
topic: observability
difficulty: hard
status: inbox
tags: [backend, production]
---

# Observability

## Definition
The ability to understand a system's internal state from its external outputs — the **three pillars** of **logs**, **metrics**, and **traces**, correlated so a single request can be reconstructed across services. Distinct from *monitoring* (predefined dashboards/alerts): observability is about being able to ask **new** questions of a live system.

## Why it matters
You can't operate or debug what you can't see. Observability turns incidents from guesswork into investigation, and defining good SLIs/SLOs is how you alert on user pain rather than noise. It's a core senior/architect responsibility.

## How it works — the three pillars
| Pillar | What | Tool (Java) | Cost driver |
|--------|------|-------------|-------------|
| **Metrics** | numeric time series (rate, errors, latency) | Micrometer → Prometheus | label **cardinality** |
| **Logs** | structured event records | SLF4J/Logback JSON → Loki/ELK | volume |
| **Traces** | spans across services for one request | OpenTelemetry / Micrometer Tracing | sampling rate |

### Correlation is the point
Propagate a **trace id** (W3C `traceparent`) through every hop and stamp it on logs + spans. Then one id ties the metric spike → the slow span → the exact log line. Without correlation you have three disconnected data sources.

### What to measure — RED / USE
- **RED** (per request-driven service): **R**ate, **E**rrors, **D**uration (latency percentiles).
- **USE** (per resource): **U**tilization, **S**aturation, **E**rrors.
- Always track **percentiles** (p50/p95/p99/p99.9), never just averages — averages hide the tail that users feel.

### SLI / SLO / SLA + error budgets
- **SLI**: a measured indicator (e.g. % of requests < 300 ms).
- **SLO**: the target (e.g. 99.9% of requests < 300 ms/28 days).
- **SLA**: the contractual promise (+ penalties).
- **Error budget** = 1 − SLO; alert on **burn rate** (how fast you're consuming the budget), not on raw CPU — this is symptom-based alerting.

## Enterprise example
```yaml
management:
  endpoints.web.exposure.include: health,prometheus,metrics
  tracing.sampling.probability: 0.1        # sample 10% of traces (cost control)
# App emits: http_server_requests (RED), tagged with route + status; trace id in every log line via MDC
```

## Trade-offs
- More telemetry = more insight but more cost. Control it: bound metric **label cardinality** (never put user-id/order-id as a label), **sample** traces, and set log levels sensibly.

## Common mistakes (senior-level)
- Unstructured logs with no trace/correlation id → can't follow a request across services.
- Alerting on causes (CPU/memory) instead of symptoms (SLO burn) → noisy, misses real pain.
- Reporting averages instead of percentiles → tail latency invisible.
- High-cardinality metric labels → Prometheus cardinality explosion / cost blowup.
- 100% trace sampling in prod (cost) or 0% (blind).

## Interview questions (staff+)
- The three pillars and how correlation ties them together.
- SLI vs SLO vs SLA; what is an error budget and burn-rate alerting?
- Why percentiles over averages? What is RED vs USE?
- Why is metric cardinality dangerous, and what must never be a label?
- How would you debug a p99 latency spike across services?

## Related concepts
- [[Actuator]]
- [[Circuit-Breaker]]
- [[Timeout]]
