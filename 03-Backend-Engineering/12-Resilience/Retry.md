---
type: concept
domain: backend
topic: resilience
difficulty: medium
status: inbox
tags: [backend]
---

# Retry

## Definition
Re-attempting a failed operation — but only for **transient**, **retryable** errors, only on **idempotent** operations, with **bounded** attempts and **exponential backoff + jitter**. Naive retries are an anti-pattern that amplifies outages.

## Why it matters
Transient blips (brief timeouts, 503s, leader elections) are common in distributed systems; smart retries raise success rates. Dumb retries cause **retry storms** — a struggling service gets hammered by synchronized retries and never recovers. Knowing *when not to retry* is the senior signal.

## How it works — the mechanism
- **Only retry idempotent operations** — or non-idempotent ones guarded by an [[Idempotency]] key. Retrying a non-idempotent `POST /charge` without a key = double charge.
- **Only retry transient errors**: timeouts, connection resets, 502/503/429 (respect `Retry-After`). **Never retry** 400/401/403/404/422 — the result won't change.
- **Exponential backoff**: `delay = base × 2^attempt`, capped.
- **Jitter** (randomize the delay) is essential: without it, all failed callers retry at the same instant → synchronized thundering herd. Full jitter: `random(0, base × 2^attempt)`.
- **Bounded attempts + overall deadline** — give up to a fallback (fits the [[Timeout]] budget).

```text
attempt 1 -> fail -> wait ~random(0,100ms)
attempt 2 -> fail -> wait ~random(0,200ms)
attempt 3 -> fail -> fallback / propagate
```

## Enterprise example
```java
RetryConfig cfg = RetryConfig.custom()
    .maxAttempts(3)
    .intervalFunction(IntervalFunction.ofExponentialRandomBackoff(100, 2.0))  // backoff + jitter
    .retryOnException(e -> e instanceof TimeoutException || e instanceof RetryableStatus)
    .build();
// Order matters: retry INSIDE the circuit breaker so repeated failures open it (see below)
```

## Retry + circuit breaker + timeout (the stack)
Compose them: **timeout** bounds each try, **retry** handles transient failures, **circuit breaker** stops retrying entirely when a dependency is broadly down ([[Circuit-Breaker]]). Get the layering right — retries should count toward opening the breaker so a hard-down dependency isn't retried into the ground.

## Trade-offs
- Improves success rate for transient faults, but adds latency and load; unbounded/no-jitter retries worsen outages.
- Client-side retries multiply load; consider server-side idempotency + client budgets (retry budgets cap the % of traffic that is retries).

## Common mistakes (senior-level)
- Retrying non-idempotent writes without an idempotency key → duplicates.
- Retrying non-retryable 4xx errors.
- No jitter → synchronized retry storms.
- Unbounded retries / no overall deadline.
- Retrying *and* the caller retrying *and* the LB retrying → multiplicative amplification.

## Interview questions (staff+)
- When is it safe to retry, and which errors are retryable?
- Why is jitter essential? What is a retry storm?
- How do retry, timeout, and circuit breaker compose (and in what order)?
- How do you retry a non-idempotent operation safely?

## Related concepts
- [[Timeout]]
- [[Circuit-Breaker]]
- [[Idempotency]]
- [[Rate-Limiting]]
