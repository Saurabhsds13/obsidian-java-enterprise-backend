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
A distributed, partitioned, replicated **commit log** for high-throughput event streaming. Unlike a traditional broker, messages are **retained** (by time/size) and consumers track their own **offset**, enabling replay and multiple independent consumer groups over the same data.

## Why it matters
Kafka is the backbone of event-driven architecture — decoupling services, buffering load, enabling replay and stream processing. Architect depth lives in partitions/ordering, delivery semantics, consumer-group rebalancing, and the exactly-once nuance.

## How it works — the mechanism
- **Topic → partitions**: a partition is the unit of parallelism *and* ordering. Order is guaranteed **only within a partition**. The producer's **partition key** decides placement (`hash(key) % partitions`) → same key → same partition → ordered per entity.
- **Producers**: `acks=all` + `min.insync.replicas` for durability; `enable.idempotence=true` prevents producer-retry duplicates.
- **Consumer groups**: each partition is consumed by exactly one consumer in a group → parallelism up to partition count. Adding consumers beyond partitions leaves some idle.
- **Offsets**: consumer commits its position. **Commit after processing** for at-least-once; committing before → at-most-once (may lose).
- **Replication**: each partition has a leader + followers (ISR = in-sync replicas); durability = `acks=all` + `min.insync.replicas ≥ 2`.
- **Rebalancing**: when consumers join/leave, partitions are reassigned; long processing between polls (`max.poll.interval.ms`) triggers a rebalance storm.

## Delivery semantics
| Semantic | How | Reality |
|----------|-----|---------|
| At-most-once | commit before processing | may lose |
| **At-least-once** | commit after processing | may dup → **idempotent consumer** ([[Idempotency]]) |
| Exactly-once (EOS) | idempotent producer + transactions (read-process-write within Kafka) | works **inside Kafka**; external side effects still need idempotency |

## Enterprise example — reliable produce + safe consume
```java
// Producer: durable + no producer-retry dups
props.put(ACKS_CONFIG, "all");
props.put(ENABLE_IDEMPOTENCE_CONFIG, true);
producer.send(new ProducerRecord<>("payments", order.id(), event));  // key=orderId -> ordered per order

// Consumer: process THEN commit; make the handler idempotent (dedup on event id)
@KafkaListener(topics = "payments", groupId = "billing")
public void onEvent(PaymentEvent e) {
    if (processed.putIfAbsent(e.id(), true) == null) handle(e);   // dedup
}   // offsets committed after successful processing
```

## Reliability patterns
- **Transactional outbox** to publish reliably with the DB write ([[Message-Queues]]).
- **Dead-letter topic** for poison messages after N retries (so one bad message doesn't stall a partition).
- Monitor **consumer lag** (log-end offset − committed offset) as the key health metric ([[Kafka-Consumer-Lag]]).

## Trade-offs
- Huge throughput, durability, replay — but only per-partition ordering, eventual consistency, and real operational complexity (partitions, ISR, rebalances).

## Common mistakes (senior-level)
- Assuming global ordering (it's per-partition).
- Non-idempotent consumers under at-least-once → duplicate side effects ([[How-would-you-prevent-duplicate-Kafka-processing]]).
- Choosing partition count too low (caps consumer parallelism) or too high (overhead, more rebalancing).
- Long per-message processing exceeding `max.poll.interval.ms` → repeated rebalances.
- Treating Kafka EOS as end-to-end exactly-once for external systems (it isn't).

## Interview questions (staff+)
- How is ordering guaranteed, and what does global ordering cost?
- Walk the delivery semantics; why is external exactly-once effectively impossible?
- How do consumer groups + partitions determine parallelism?
- What does `acks=all` + `min.insync.replicas` guarantee?
- How do you prevent duplicate processing? ([[How-would-you-prevent-duplicate-Kafka-processing]])

## Related concepts
- [[Idempotency]]
- [[Message-Queues]]
- [[Eventual-Consistency]]
- [[Kafka-Consumer-Lag]]
