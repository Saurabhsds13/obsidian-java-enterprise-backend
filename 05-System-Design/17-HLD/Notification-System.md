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
Deliver notifications across channels (push, email, SMS, in-app) reliably at scale.

## 2. Requirements
### Functional
- Send single and bulk notifications; support multiple channels and templates; user preferences/opt-out.
### Non-functional
- High throughput, at-least-once delivery, resilience to provider outages, no duplicates from the user's view.
### Scale assumptions
- Millions of notifications/day with spikes.

## 3. API design
```text
POST /api/v1/notifications { userId, template, channel, data }
```

## 4. Data model
`notifications(id, user_id, channel, status, template, payload, created_at)`, `user_prefs(...)`.

## 5. High-level architecture
API → [[Message-Queues|queue]] → channel workers → provider adapters (APNs/FCM, email, SMS). Preference + template services on the side.

## 6. Component responsibilities
- Ingestion API validates and enqueues.
- Workers per channel with retries/backoff.
- Provider adapters isolate third-party APIs behind [[Circuit-Breaker]] + [[Timeout]].

## 7. Database choice
Relational for state/audit; fast store for dedup keys ([[Redis]]).

## 8. Cache strategy
Cache templates and user preferences.

## 9. Messaging strategy
Durable queue decouples spikes; dead-letter queue for repeated failures; [[Kafka]] for high volume.

## 10. Scaling strategy
Scale workers per channel independently; partition by user.

## 11. Failure scenarios
Provider down → retry/backoff, breaker open, fall back to alternate channel; queue buffers backlog.

## 12. Consistency decisions
At-least-once + [[Idempotency]] (dedup key) so users don't see duplicates.

## 13. Security considerations
Authn on send API, PII handling, respect opt-outs, no secrets in payloads.

## 14. Observability
Delivery rate, per-provider latency/error, queue lag ([[Observability]]).

## 15. Bottlenecks
Provider rate limits; template rendering; hot users.

## 16. Trade-offs
At-least-once + dedup vs elusive exactly-once; per-channel scaling complexity.

## 17. Interview discussion points
Dedup design, prioritization (OTP vs marketing), rate limiting to providers, fan-out for bulk.

## Related concepts
- [[Message-Queues]]
- [[Idempotency]]
- [[Circuit-Breaker]]
- [[Kafka]]
