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
Designing a limiter that enforces per-client quotas **consistently across a distributed fleet**, so a client's limit holds regardless of which node serves each request. Extends the algorithm choices in [[Rate-Limiting]] to the multi-node, high-throughput setting.

## Why it matters
Per-node limiting is wrong at scale (N nodes → N× the intended limit). A correct distributed limiter is essential for abuse protection, fairness, quota/billing tiers, and protecting downstreams — and it's a very common system-design question.

## How it works — the algorithms
| Algorithm | Behavior | Memory | Notes |
|-----------|----------|--------|-------|
| **Fixed window** | count per wall-clock window | O(1) | boundary burst: 2× at the edge |
| **Sliding window log** | timestamps of each request | O(requests) | exact, memory-heavy |
| **Sliding window counter** | weighted prev+current window | O(1) | good accuracy/cost balance |
| **Token bucket** | tokens refill at rate r, burst = capacity | O(1) | allows bursts, most common |
| **Leaky bucket** | queue drains at constant rate | O(1) | smooths output |

### Making it distributed
- Keep counters in a **shared store** ([[Redis]]) so all nodes agree.
- The check-and-update must be **atomic** — do it in a **Redis Lua script** (read + decide + write in one round trip, no race) or `INCR`+`EXPIRE`.
- Return **429 Too Many Requests** with a **`Retry-After`** header.

```lua
-- token bucket in one atomic Lua call: refill by elapsed time, then try to take 1
-- returns 1 (allowed) or 0 (limited)
```

### Accuracy vs cost at extreme scale
A central Redis call per request adds latency and a hot dependency. Alternative: **local token buckets** on each node, periodically synced/allocated a share of the global budget — approximate but cheap and resilient (works if Redis blips). Choose exact-central vs approximate-local by requirements.

## Enterprise example — atomic Redis counter
```text
key = rl:{userId}:{windowStart}
Lua: count = INCR key; if count == 1 then PEXPIRE key windowMs end; return count
allow if count <= limit else 429 + Retry-After
```

## Where to enforce
- **API gateway**: coarse, global, cheap first line.
- **Per service**: fine-grained, resource-specific.
- **Tiered**: different limits per plan (free vs enterprise), per endpoint cost.

## Trade-offs
- Central store = accurate global limits but latency + a dependency (needs a fail-open/closed policy if Redis is down).
- Local = fast and resilient but only approximate.
- Fixed window is cheapest but bursts at boundaries; sliding/token bucket cost slightly more for correctness.

## Common mistakes (senior-level)
- Per-instance counters assumed to be global → N× the limit.
- Non-atomic check-then-increment → over-admission under concurrency.
- Fixed windows enabling 2× bursts at the boundary.
- No `Retry-After` → clients retry aggressively and make it worse.
- No decision for "what if Redis is down" (fail-open vs fail-closed).

## Interview questions (staff+)
- Design a distributed rate limiter; why must the counter update be atomic?
- Token bucket vs sliding window vs fixed window — trade-offs.
- Central Redis vs local buckets at 1M req/s — which and why?
- Fail-open or fail-closed when the limiter store is unavailable?

## Related concepts
- [[Rate-Limiting]]
- [[Redis]]
- [[Distributed-Locks]]
- [[Load-Balancing]]
