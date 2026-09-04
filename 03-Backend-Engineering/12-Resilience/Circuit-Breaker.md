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
A resilience pattern that monitors calls to a dependency and, when failures exceed a threshold, **trips open** to fail fast (or serve a fallback) for a cooldown — instead of piling up doomed calls. After cooldown it probes recovery before fully closing.

## Why it matters
It's the primary defense against **cascading failure**: it stops a struggling downstream from consuming the caller's threads/connections and gives the downstream room to recover. A staple of every resilient-architecture interview.

## How it works — the state machine
```
CLOSED  --failure rate > threshold-->  OPEN
  ^                                      |
  |                              (after waitDuration)
  |                                      v
  +----- success ----  HALF-OPEN  ---- failure --> OPEN
                    (allow N trial calls)
```
- **Closed**: calls flow; a sliding window (count- or time-based) tracks failure %.
- **Open**: calls **fail fast** (or fallback) immediately for `waitDurationInOpenState` — no thread is spent on the doomed dependency.
- **Half-Open**: a limited number of trial calls test recovery; enough successes → Closed, any failure → Open again.

### Related patterns (usually combined)
- **Bulkhead**: isolate each dependency's thread/connection pool so one slow dependency can't drain the shared pool ([[ExecutorService]]).
- **Fallback**: cached/last-known value, default, or degraded response when open.
- **Slow-call detection**: modern breakers (Resilience4j) also trip on calls that are *slow* (not just failing), catching the "slow downstream" case ([[Timeout]]).

## Enterprise example
```java
@CircuitBreaker(name = "pricing", fallbackMethod = "cachedPrice")
@TimeLimiter(name = "pricing")          // enforce a timeout
@Bulkhead(name = "pricing")             // isolate the pool
public CompletableFuture<Price> getPrice(String id) { return pricingClient.getAsync(id); }

private CompletableFuture<Price> cachedPrice(String id, Throwable t) {
    return CompletableFuture.completedFuture(cache.lastKnown(id));   // graceful degradation
}
```

## Tuning (from metrics, not guesses)
- **Failure-rate threshold** (e.g. 50%) and **window size** — too sensitive → flapping; too lax → never trips.
- **waitDuration** — long enough for the dependency to recover, short enough to restore service quickly.
- Base all of these on observed error rates and recovery times ([[Observability]]).

## Trade-offs
- Fails fast to protect the caller, but returns errors/fallbacks while open (a deliberate availability trade).
- Adds tuning burden and can mask a persistent problem if the fallback is "good enough" — alert on breaker-open events.

## Common mistakes (senior-level)
- No breaker → cascading failure when a dependency degrades.
- Thresholds too sensitive (flapping open/closed) or too lax (never opens).
- No meaningful fallback (open state just returns 500s).
- Only detecting failures, not **slow** calls (a slow dependency still exhausts threads).
- One shared thread pool for all dependencies (no bulkhead) → one slow dep drains everything.

## Interview questions (staff+)
- Walk the three states and the half-open probe.
- How does a breaker prevent cascading failure?
- Why add bulkheads and slow-call detection alongside it?
- How do you tune the threshold and wait duration?
- How do retry, timeout, and breaker compose?

## Related concepts
- [[Timeout]]
- [[Retry]]
- [[Rate-Limiting]]
- [[Observability]]
