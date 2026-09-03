---
type: interview
domain: system-design
topic: idempotency
difficulty: hard
status: inbox
tags: [interview, system-design]
---

# How would you design an idempotent payment API?

## Question
> How do you ensure a payment endpoint never double-charges when clients retry?

## Short answer
Require an idempotency key; store the key with the result atomically; a repeat with the same key returns the stored result instead of charging again.

## Detailed answer
Clients send `Idempotency-Key: <uuid>` with the charge. The server records the key (unique constraint or [[Redis]] `SETNX`) **atomically** with creating the payment, so concurrent retries can't both execute. A repeat returns the recorded outcome. Persist a payment **state machine** (INITIATED → AUTHORIZED → CAPTURED/FAILED) so illegal transitions are rejected. For unknown provider results (timeout), reconcile via status query/webhook rather than blindly retrying. Combine with strong consistency ([[ACID]]) and the outbox pattern for events.

## Example
```java
idempotencyStore.find(key)
  .map(ResponseEntity::ok)
  .orElseGet(() -> { var r = service.charge(req); idempotencyStore.save(key, r); return ok(r); });
```

## Production relevance
Standard for payments and any at-least-once side effect ([[Kafka]]).

## Common mistake
Non-atomic check-then-write, or scoping the key so a different body reuses the same key.

## Follow-up questions
- Where do you store keys and with what TTL?
- How do you handle a timeout with unknown provider outcome?

## Related concepts
- [[Idempotency]]
- [[Payment-System]]
- [[ACID]]
