---
type: concept
domain: system-design
topic: messaging
difficulty: hard
status: inbox
tags: [system-design]
---

# Message Queues

## Definition
Infrastructure for **asynchronous** communication: producers hand messages to a broker, which stores and delivers them to consumers, decoupling the two in time and load. Two families: **traditional brokers** (RabbitMQ, SQS — queue/exchange, message deleted after ack) and **log-based** systems ([[Kafka]] — append-only partitioned log, messages retained and replayable).

## Why it matters
Queues are the backbone of resilient, scalable, event-driven architecture — they absorb spikes, decouple services, and enable retries. The senior depth is in delivery semantics, ordering, and the failure patterns (duplicates, poison messages, backlog).

## How it works — the mechanism

### Delivery semantics (the core interview axis)
| Semantic | Guarantee | Cost | Reality |
|----------|-----------|------|---------|
| At-most-once | may lose, never dup | cheapest | fire-and-forget metrics |
| **At-least-once** | never lose, may dup | ack after processing | **the common default → consumers must be idempotent** |
| Exactly-once | no loss, no dup | expensive/limited | only within a closed system (e.g. Kafka transactions producer→topic); external side effects still need [[Idempotency]] |

"Exactly-once delivery" to arbitrary external systems is effectively impossible; you achieve **effectively-once** = at-least-once + idempotent consumer.

### Ordering
- Kafka guarantees order **within a partition** only; key-based partitioning keeps a given entity's events ordered. Global ordering means one partition → no parallelism.
- Traditional queues generally don't guarantee order across concurrent consumers.

### Reliability patterns
- **Dead-letter queue (DLQ)**: after N failed retries, route the poison message aside so it stops blocking the partition/consumer.
- **Transactional outbox**: write the domain change *and* an outbox row in one DB transaction; a relay publishes the outbox to the broker — eliminates the dual-write problem (DB commit but publish fails, or vice versa).
- **Consumer groups**: scale consumers up to the partition count for throughput.

## Enterprise example — outbox to avoid dual-write inconsistency
```java
@Transactional
public void placeOrder(Order o) {
    orderRepo.save(o);
    outboxRepo.save(new OutboxEvent("OrderPlaced", o.id()));   // same tx as the state change
}
// A separate relay polls outbox -> publishes to Kafka -> marks sent (at-least-once)
```

## Queue vs pub/sub vs log
| Model | Fan-out | Replay | Example |
|-------|---------|--------|---------|
| Queue (point-to-point) | one consumer per msg | no | SQS, Rabbit queue |
| Pub/Sub | all subscribers | usually no | SNS, Rabbit fanout |
| Log | consumer groups, offset-based | **yes** (retention) | Kafka |

## Trade-offs
- Advantages: decoupling, spike absorption, retries, resilience to downstream outages, replay (log).
- Disadvantages: eventual consistency, duplicate/ordering handling, extra infra to operate, harder end-to-end tracing.

## Common mistakes (senior-level)
- Assuming exactly-once and skipping [[Idempotency]] → duplicate side effects.
- Assuming global ordering (it's per-partition in Kafka).
- No DLQ → one poison message stalls a partition and grows lag.
- Dual-write (DB + publish) without the outbox pattern → lost or ghost events.
- Unbounded consumer concurrency hammering a downstream (still need backpressure/rate limits).

## Interview questions (staff+)
- Compare the three delivery semantics; why is external exactly-once effectively impossible?
- How is ordering guaranteed, and what does global ordering cost?
- Explain the transactional outbox and the problem it solves.
- What does a DLQ do and when does a message land there?
- Queue vs pub/sub vs log — pick one for a given scenario.

## Related concepts
- [[Kafka]]
- [[Idempotency]]
- [[Eventual-Consistency]]
