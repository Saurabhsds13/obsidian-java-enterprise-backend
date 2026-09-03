---
type: concept
domain: system-design
topic: rate-limiting
difficulty: hard
status: inbox
tags: [system-design]
---

# Rate Limiting (Design)

## Definition
Designing a rate limiter that works across a distributed fleet, enforcing per-client quotas consistently regardless of which node handles a request.

## Why it matters
Per-node limiting is inaccurate at scale. A distributed limiter enforces true global quotas for abuse protection, fairness, and billing tiers.

## How it works
- Keep counters in a shared store ([[Redis]]) so all nodes agree.
- Algorithm choices (see [[Rate-Limiting]]): token bucket (bursts), sliding window log/counter (accuracy).
- Make the check-and-update **atomic** (Redis Lua script or `INCR` + `EXPIRE`) to avoid races.
- Return `429` with `Retry-After`.

```text
key = "rl:{userId}:{windowStart}"
atomic: count = INCR key; if count == 1 then EXPIRE key windowSize
allow if count <= limit
```

## Production usage
Enforce at the gateway for coarse limits and per-service for fine ones. Tier limits by plan. For huge scale, use local token buckets synced periodically to reduce Redis load (approximate but cheaper).

## Trade-offs
- Central store = accurate but a dependency and latency add; local = fast but approximate.

## Common mistakes
- Non-atomic counter updates (over-admitting under concurrency).
- Fixed windows causing double-rate bursts at boundaries.

## Interview questions
- Design a distributed rate limiter. (see [[How-would-you-design-a-distributed-rate-limiter]])
- How do you keep it accurate under concurrency?

## Related concepts
- [[Rate-Limiting]]
- [[Redis]]
- [[Distributed-Locks]]
