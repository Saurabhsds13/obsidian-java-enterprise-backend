---
type: interview
domain: system-design
topic: rate-limiting
difficulty: hard
status: inbox
tags: [interview, system-design]
---

# How would you design a distributed rate limiter?

## Question
> Design a rate limiter that enforces per-client limits across many application instances.

## Short answer
Keep counters in a shared store (Redis) updated atomically, using token bucket or sliding window, and return 429 with Retry-After when the limit is exceeded.

## Detailed answer
Per-node counters can't enforce a global limit, so centralize state in [[Redis]]. Use an atomic operation (Lua script or `INCR`+`EXPIRE`) to avoid races. Choose an algorithm: token bucket (allows bursts), sliding window (accurate, avoids fixed-window boundary bursts). For extreme scale, use local buckets synced periodically (approximate but cheaper). Tier limits by plan and enforce at the gateway plus per service. See [[Rate-Limiting-Design]].

## Example
```text
key = rl:{userId}:{window}
count = INCR key; if count == 1 then EXPIRE key windowSize
allow if count <= limit else 429 + Retry-After
```

## Production relevance
Protects capacity, ensures fairness, enforces billing tiers.

## Common mistake
Non-atomic check-then-increment (over-admits under concurrency); fixed windows doubling the effective rate at boundaries.

## Follow-up questions
- Token bucket vs sliding window?
- How do you make it fault-tolerant if Redis is down?

## Related concepts
- [[Rate-Limiting-Design]]
- [[Rate-Limiting]]
- [[Redis]]
