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
An upper bound on how long a call waits before giving up, freeing the caller's resources.

## Why it matters
Missing timeouts are the classic cause of cascading failure: a slow downstream ties up threads/connections until the whole service stalls.

## How it works
- Set **connect** and **read** timeouts on every remote client (HTTP, DB, cache).
- A timeout should be shorter than the caller's own deadline so retries/fallbacks have time.

```java
RestClient client = RestClient.builder()
    .requestFactory(new SimpleClientHttpRequestFactory() {{
        setConnectTimeout(200);   // ms
        setReadTimeout(800);      // ms
    }})
    .build();
```

## Production usage
Every outbound dependency must have a timeout. Combine with [[Retry]] (bounded), [[Circuit-Breaker]], and a fallback. Budget timeouts across a call chain so the total stays within the client SLA.

## Trade-offs
- Too short → false failures; too long → resource exhaustion. Base values on measured latency percentiles.

## Common mistakes
- Relying on default (often infinite) timeouts.
- Timeouts longer than the request's own deadline.

## Interview questions
- What happens when a downstream service becomes slow? (see [[What-happens-when-a-downstream-service-becomes-slow]])
- How do you choose timeout values?

## Related concepts
- [[Retry]]
- [[Circuit-Breaker]]
- [[Rate-Limiting]]
