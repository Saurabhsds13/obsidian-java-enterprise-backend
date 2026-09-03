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
Re-attempting a failed operation, ideally only for transient errors, with backoff and a bounded number of attempts.

## Why it matters
Transient failures (blips, brief timeouts) are common in distributed systems; smart retries improve reliability. Dumb retries cause **retry storms** that amplify outages.

## How it works
- Retry only **idempotent** or idempotency-keyed operations ([[Idempotency]]).
- Use **exponential backoff + jitter** to avoid synchronized retry waves.
- Cap attempts and total time; give up to a fallback.

```text
attempt 1 -> fail -> wait ~100ms (+jitter)
attempt 2 -> fail -> wait ~200ms (+jitter)
attempt 3 -> fail -> fallback / propagate error
```

## Production usage
Pair with [[Timeout]] and [[Circuit-Breaker]] (stop retrying when the breaker is open). Never retry non-idempotent writes without an idempotency key.

## Trade-offs
- Improves success rate but adds latency and load; unbounded retries worsen outages.

## Common mistakes
- Retrying non-idempotent operations (duplicate side effects).
- No jitter (thundering herd).

## Interview questions
- When is it safe to retry?
- Why add jitter to backoff?

## Related concepts
- [[Timeout]]
- [[Circuit-Breaker]]
- [[Idempotency]]
