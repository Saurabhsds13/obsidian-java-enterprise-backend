---
type: concept
domain: backend
topic: resilience
difficulty: medium
status: inbox
tags: [backend]
---

# Timeout

## Definition
An upper bound on how long an operation waits before abandoning it and freeing the caller's resources (thread, connection). Every remote interaction has multiple timeouts: **connect**, **read/socket**, **request/overall**, and **pool-acquire**.

## Why it matters
Missing or infinite timeouts are the root cause of most cascading failures: a slow dependency ties up threads and connections until the whole service stalls. Timeouts are the *first* resilience control — retries and breakers are meaningless without them.

## How it works — the layers of timeout
| Timeout | Bounds | Typical |
|---------|--------|---------|
| connect | TCP handshake | 100–500 ms |
| read / socket | waiting for bytes after connect | based on p99 service time |
| request / overall | whole call incl. retries | must fit the caller's deadline |
| pool-acquire | waiting for a free connection ([[Connection-Pooling]]) | fail fast (1–3 s) |

### Timeout budgets across a call chain
A request has an end-to-end **deadline** (say 1 s at the edge). Each downstream must get a *smaller* budget so there's time to react (retry/fallback). If service A (1 s budget) calls B, B's timeout must be < 1 s minus A's own work and any planned retry — otherwise A times out to the user while B is still trying. Propagate deadlines (e.g. a `Deadline`/`X-Request-Timeout` header, gRPC deadlines).

## Enterprise example
```java
RestClient client = RestClient.builder()
    .requestFactory(ClientHttpRequestFactorySettings.DEFAULTS
        .withConnectTimeout(Duration.ofMillis(200))
        .withReadTimeout(Duration.ofMillis(800)))   // < caller's ~1s deadline
    .build();
// pair with a CompletableFuture.orTimeout(...) for the overall budget (see [[CompletableFuture]])
```

## Choosing values (from data, not vibes)
Base read timeouts on the dependency's **measured p99/p99.9**, not the average — set it a bit above p99 so you don't kill healthy-but-slow calls, but low enough to shed a truly stuck one. Re-tune as latency shifts.

## Trade-offs
- Too short → false failures, wasted retries, worse user experience.
- Too long → resource exhaustion, cascading failure.
- No universal value — derive per dependency from latency percentiles.

## Common mistakes (senior-level)
- Relying on library defaults (often **infinite** for read timeouts — e.g. default JDBC/socket).
- A downstream timeout ≥ the caller's own deadline (caller gives up first; work wasted).
- Timing out on connect but not read (or vice versa).
- Not bounding pool-acquire wait → threads pile up invisibly.

## Interview questions (staff+)
- What happens when a downstream gets slow with no timeout? ([[What-happens-when-a-downstream-service-becomes-slow]])
- Explain timeout budgets across a call chain and deadline propagation.
- How do you choose a read-timeout value?
- Which distinct timeouts exist on an HTTP+DB call?

## Related concepts
- [[Retry]]
- [[Circuit-Breaker]]
- [[Rate-Limiting]]
- [[Connection-Pooling]]
