---
type: concept
domain: backend
topic: kafka
difficulty: hard
status: inbox
tags: [backend]
---

# Kafka

## Definition
A distributed, partitioned, replicated commit log used for high-throughput event streaming and asynchronous messaging.

## Why it matters
Kafka is the backbone of event-driven architectures: decoupling services, buffering load, and enabling replay.

## How it works
- **Topic** → split into **partitions** (unit of parallelism and ordering). Order is guaranteed **within** a partition only.
- **Producers** write; the partition key decides placement (same key → same partition → ordered).
- **Consumer groups**: each partition is consumed by one consumer in a group; scaling consumers up to the partition count increases throughput.
- **Offsets** track progress; **replication** provides durability.
- **Delivery**: at-least-once by default → consumers must be idempotent.

```text
Topic: payments  (6 partitions)
key=orderId -> same partition -> ordered per order
Consumer group "billing": 6 consumers, one per partition
```

## Production usage
Design for **at-least-once + idempotent consumers** ([[Idempotency]]); use the **outbox pattern** to publish reliably with the DB transaction; route poison messages to a dead-letter topic. Monitor **consumer lag**.

## Trade-offs
- High throughput and durability, but only per-partition ordering and eventual consistency; operational complexity.

## Common mistakes
- Assuming global ordering.
- Non-idempotent consumers causing duplicate side effects.
- Ignoring consumer lag until it's a backlog.

## Interview questions
- How do you prevent duplicate Kafka processing? (see [[How-would-you-prevent-duplicate-Kafka-processing]])
- How is ordering guaranteed?

## Related concepts
- [[Idempotency]]
- [[Message-Queues]]
- [[Eventual-Consistency]]
