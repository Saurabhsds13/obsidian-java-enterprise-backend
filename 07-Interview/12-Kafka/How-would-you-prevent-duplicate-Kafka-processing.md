---
type: interview
domain: backend
topic: kafka
difficulty: hard
status: inbox
tags: [interview, backend]
---

# How would you prevent duplicate Kafka processing?

## Question
> Kafka is at-least-once, so a consumer may see a message twice. How do you avoid duplicate side effects?

## Short answer
Make processing idempotent — dedupe on a business key, or make writes naturally idempotent — rather than chasing exactly-once delivery.

## Detailed answer
Since redelivery is expected, design the consumer so reprocessing is safe:
- **Idempotency key / dedup store**: record processed message/business IDs (DB unique constraint or [[Redis]] `SETNX` with TTL); skip if already seen.
- **Idempotent writes**: upserts and state transitions that are safe to repeat (e.g. "set status = PAID").
- **Transactional outbox + inbox**: consume and record processing in one DB transaction.
- Commit offsets only after successful processing; route poison messages to a dead-letter topic.

## Example
```java
if (processed.putIfAbsent(msg.id(), true) == null) {
    handle(msg);   // first time only
}
```

## Production relevance
This is the standard approach for payments, orders, and any side-effecting consumer. Pairs with [[Idempotency]].

## Common mistake
Relying on "exactly-once" as a silver bullet; it's limited and doesn't cover external side effects.

## Follow-up questions
- Where do you store dedup keys and for how long?
- How does the outbox pattern help?

## Related concepts
- [[Kafka]]
- [[Idempotency]]
- [[Redis]]
