---
type: system-design
level: HLD
domain: system-design
difficulty: hard
status: inbox
tags: [system-design]
---

# Notification System

## 1. Problem
Deliver notifications reliably across channels (push, email, SMS, in-app) at scale, respecting user preferences and provider limits.

## 2. Requirements
### Functional
- Send single + bulk notifications; multiple channels + templates; user preferences/opt-out; prioritization (OTP > marketing).
### Non-functional
- High throughput; at-least-once delivery with **user-visible de-duplication**; resilient to provider outages; prioritized delivery.

### Capacity estimation
```
10M notifications/day ≈ 115/s average, peak ~10× ≈ 1,200/s
Fan-out (bulk campaign to 5M users) -> bursty millions enqueued quickly -> queue must buffer
Provider limits (e.g. SMS ~200/s per account) -> must SHAPE outbound to provider rate
Store dedup keys 24-72h in Redis; audit rows in relational store
```

## 3. API design
```text
POST /api/v1/notifications {userId, template, channel?, data, priority}
POST /api/v1/notifications:bulk {segment, template, data}
```

## 4. Data model
`notifications(id, user_id, channel, status, template, payload, created_at)`, `user_prefs(user_id, channel_opt_in...)`, dedup keys in Redis.

## 5. High-level architecture
Ingestion API → **priority [[Message-Queues|queues]]** (one per channel/priority) → channel workers → provider adapters (APNs/FCM, email, SMS) behind [[Circuit-Breaker]] + [[Timeout]]. Preference + template services alongside.

## 6. Component responsibilities
- **Ingestion**: validate, check prefs, enqueue by priority.
- **Channel workers**: pull, render template, call provider, retry/backoff, respect provider rate ([[Rate-Limiting-Design]]).
- **Provider adapters**: isolate third-party APIs; DLQ on repeated failure.

## 7. Database choice
Relational for state/audit; [[Redis]] for dedup keys + rate counters.

## 8. Cache strategy
Cache templates and user preferences (invalidate on change).

## 9. Messaging strategy
Durable queue decouples spikes; **separate queues per priority** so OTPs aren't stuck behind a marketing blast; DLQ for poison messages; [[Kafka]] for volume + replay.

## 10. Scaling strategy
Scale workers **per channel independently** (SMS is provider-limited; push scales freely). Partition by user for ordering where needed.

## 11. Failure scenarios
Provider down → retry with backoff, breaker opens, optionally fall back to an alternate channel; queue buffers the backlog; DLQ isolates poison messages.

## 12. Consistency decisions
At-least-once + **idempotency/dedup key** ([[Idempotency]]) so a user never sees duplicates despite retries and redelivery ([[Message-Queues]]).

## 13. Security considerations
AuthN on send API, PII minimization in payloads, strict opt-out enforcement, secrets in a manager (`YOUR_API_KEY`).

## 14. Observability
Delivery/failure rate per channel+provider, provider latency, **queue lag per priority**, dedup hit rate ([[Observability]]).

## 15. Bottlenecks
Provider rate limits (shape outbound), template rendering, hot users; bulk fan-out spikes.

## 16. Trade-offs
At-least-once + dedup (simple, robust) vs elusive exactly-once; per-channel/priority queues add operational complexity but protect latency-critical traffic.

## 17. Interview discussion points
Dedup design + key TTL, prioritization (dedicated queues vs priority field), outbound rate shaping to providers, bulk fan-out strategy, retry/backoff + DLQ policy.

## Related concepts
- [[Message-Queues]]
- [[Idempotency]]
- [[Circuit-Breaker]]
- [[Rate-Limiting-Design]]
- [[Kafka]]
