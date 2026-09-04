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
Restricting how many requests a client may make in a time window, to protect capacity, ensure fairness, and enforce quotas/tiers. Distinct from **throttling** (slowing) and **load shedding** (dropping low-priority work under stress).

## Why it matters
Rate limits shield services from abuse, runaway clients, and traffic spikes, and back billing tiers. The senior depth is in the algorithm trade-offs and making limits correct across a fleet (see [[Rate-Limiting-Design]]).

## How it works — the algorithms
| Algorithm | Behavior | Memory | Boundary burst |
|-----------|----------|--------|----------------|
| Fixed window | count per wall-clock window | O(1) | yes (2× at edge) |
| Sliding window log | timestamp per request | O(n) | no (exact) |
| Sliding window counter | weighted prev+current window | O(1) | minimal |
| **Token bucket** | tokens refill at rate r, capacity = burst | O(1) | allows controlled bursts |
| Leaky bucket | queue drains at constant rate | O(1) | smooths output |

**Token bucket** is the common default: it permits short bursts (up to capacity) while enforcing a sustained rate — matching real client behavior.

## Enterprise example — token bucket + 429
```java
if (!rateLimiter.tryAcquire(userId)) {
    return ResponseEntity.status(429)
        .header("Retry-After", "1")
        .body(new ApiError("rate_limited", "Try again shortly"));
}
```
Always return **429 Too Many Requests** with **`Retry-After`** so well-behaved clients back off instead of hammering.

## Where to enforce (defense in depth)
- **Edge / API gateway**: coarse global limits, cheap, first line.
- **Per service / endpoint**: fine-grained, weighted by cost (a search endpoint costs more than a health check).
- **Per tier**: free vs enterprise quotas.
Distributed enforcement needs shared, atomic counters — see [[Rate-Limiting-Design]] and [[Redis]].

## Trade-offs
- Protects the service but rejects legitimate bursts if too tight; too loose and it doesn't protect.
- Per-node limits are simple but wrong at scale (N× the intended limit) → distributed counter adds a dependency.

## Common mistakes (senior-level)
- Per-instance limits assumed to be global.
- No `Retry-After` → clients retry aggressively (compounds the problem with [[Retry]]).
- Fixed windows allowing 2× bursts at the boundary.
- Rate limiting by IP only (breaks behind NAT/proxies; use API key/user + IP).
- Not differentiating endpoint cost (one limit for cheap and expensive calls).

## Interview questions (staff+)
- Token bucket vs sliding window vs fixed window — trade-offs.
- Why 429 + Retry-After, and how does it interact with client retries?
- Where do you enforce limits (edge vs service), and why both?
- How do you make limits correct across many instances? ([[Rate-Limiting-Design]])

## Related concepts
- [[Rate-Limiting-Design]]
- [[Redis]]
- [[Circuit-Breaker]]
- [[Retry]]
