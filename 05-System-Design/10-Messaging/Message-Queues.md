---
type: concept
domain: system-design
topic: messaging
difficulty: medium
status: inbox
tags: [system-design]
---

# Message Queues

## Definition
Infrastructure that lets services communicate asynchronously by passing messages through a broker, decoupling producers from consumers.

## Why it matters
Queues absorb spikes, decouple services, enable retries and buffering, and are the backbone of event-driven architecture.

## How it works
- **Producer** sends; **broker** stores; **consumer** processes at its own pace.
- **Queue** (point-to-point): each message to one consumer. **Pub/Sub**: each message to all subscribers.
- Delivery semantics: at-most-once, at-least-once (common → need [[Idempotency]]), exactly-once (hard/limited).
- [[Kafka]] is a log-based system enabling replay; RabbitMQ/SQS are classic brokers.

## Production usage
Use queues to smooth load (async work, emails, notifications), decouple services, and buffer during downstream outages. Add dead-letter queues for poison messages and monitor backlog/lag.

## Trade-offs
- Decoupling and resilience vs added infrastructure, eventual consistency, and ordering/duplication concerns.

## Common mistakes
- Assuming exactly-once and skipping idempotency.
- No dead-letter handling → stuck consumers.

## Interview questions
- When do you introduce a queue?
- Queue vs pub/sub; at-least-once vs exactly-once?

## Related concepts
- [[Kafka]]
- [[Idempotency]]
- [[Eventual-Consistency]]
