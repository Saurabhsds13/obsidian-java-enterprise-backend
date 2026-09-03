---
type: concept
domain: backend
topic: resilience
difficulty: medium
status: inbox
tags: [backend]
---

# Circuit Breaker

## Definition
A resilience pattern that stops calling a failing dependency for a cooldown period, failing fast instead of piling up doomed requests.

## Why it matters
It prevents a struggling downstream from dragging the caller down and gives the dependency time to recover — key to stopping cascading failure.

## How it works
Three states:
- **Closed**: calls flow; failures are counted.
- **Open**: failure threshold exceeded → calls fail fast (or fall back) for a cooldown.
- **Half-open**: after cooldown, a few trial calls test recovery; success → closed, failure → open.

```java
// Resilience4j
@CircuitBreaker(name = "pricing", fallbackMethod = "cachedPrice")
public Price getPrice(String id) { return pricingClient.get(id); }

private Price cachedPrice(String id, Throwable t) { return cache.lastKnown(id); }
```

## Production usage
Wrap remote calls with a breaker + [[Timeout]] + bounded [[Retry]] + fallback. Tune thresholds and window from real metrics. Expose breaker state to [[Observability]].

## Trade-offs
- Fails fast (protects the caller) but returns errors/fallbacks during open state; tuning is workload-specific.

## Common mistakes
- Thresholds too sensitive (flapping) or too lax (never trips).
- No meaningful fallback.

## Interview questions
- Explain the circuit breaker states.
- How does it prevent cascading failure?

## Related concepts
- [[Timeout]]
- [[Retry]]
- [[Rate-Limiting]]
