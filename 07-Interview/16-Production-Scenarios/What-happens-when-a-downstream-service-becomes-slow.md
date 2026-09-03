---
type: interview
domain: backend
topic: resilience
difficulty: hard
status: inbox
tags: [interview, production]
---

# What happens when a downstream service becomes slow?

## Question
> A dependency your service calls becomes slow. What happens, and how do you protect yourself?

## Short answer
Without timeouts, callers pile up waiting, exhausting threads/connections and cascading the failure. Protect with timeouts, bounded retries, circuit breakers, bulkheads, and fallbacks.

## Detailed answer
Each in-flight call holds a thread and a connection. If the downstream stalls and there's no [[Timeout]], those resources are never released; new requests queue until the pool is exhausted and the caller itself becomes unavailable — a cascading failure. Defenses:
- **Timeouts** on every remote call.
- **Circuit breaker** ([[Circuit-Breaker]]) to fail fast when errors spike.
- **Bulkheads** to isolate the dependency's thread/connection pool.
- **Bounded retries with backoff+jitter** ([[Retry]]) — never unbounded.
- **Fallback** (cached/last-known value or degraded response).

## Example
Wrap the call: timeout 500ms → breaker opens after failures → serve a cached fallback.

## Production relevance
This is the classic cascading-failure incident. Load-shedding and graceful degradation keep the core service alive.

## Common mistake
Default (infinite) timeouts and unbounded retries that amplify the outage.

## Follow-up questions
- How do you size timeouts across a call chain?
- What is a bulkhead and why does it help?

## Related concepts
- [[Timeout]]
- [[Circuit-Breaker]]
- [[Retry]]
