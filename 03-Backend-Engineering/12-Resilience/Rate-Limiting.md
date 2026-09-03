---
type: concept
domain: backend
topic: resilience
difficulty: medium
status: inbox
tags: [backend]
---

# Rate Limiting

## Definition
Restricting how many requests a client may make in a time window to protect capacity and ensure fair use.

## Why it matters
Rate limits shield services from abuse, runaway clients, and traffic spikes, and enforce quota/billing tiers.

## How it works
Common algorithms:
- **Token bucket**: tokens refill at a fixed rate; each request consumes one; allows bursts up to bucket size.
- **Leaky bucket**: smooths output at a constant rate.
- **Fixed / sliding window**: count requests per window; sliding window avoids boundary bursts.

```text
Token bucket (capacity 10, refill 5/sec):
  burst of 10 allowed, then ~5/sec sustained
```

## Production usage
Enforce at the API gateway and/or per service. For distributed limits, keep counters in [[Redis]] (atomic `INCR`/Lua) so all instances share state. Return `429 Too Many Requests` with `Retry-After`. See [[Rate-Limiting-Design]].

## Trade-offs
- Per-node limits are simple but inaccurate at scale; shared (Redis) limits are accurate but add a dependency.

## Common mistakes
- Per-instance limits assumed to be global.
- No `Retry-After`, so clients hammer harder.

## Interview questions
- How would you design a distributed rate limiter? (see [[How-would-you-design-a-distributed-rate-limiter]])
- Token bucket vs sliding window?

## Related concepts
- [[Redis]]
- [[Rate-Limiting-Design]]
- [[Circuit-Breaker]]
