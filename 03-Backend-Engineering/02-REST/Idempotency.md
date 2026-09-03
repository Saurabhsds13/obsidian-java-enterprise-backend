---
type: concept
domain: backend
topic: rest
difficulty: hard
status: inbox
tags: [backend]
---

# Idempotency

## Definition
An operation is idempotent if performing it multiple times has the same effect as performing it once. GET/PUT/DELETE are idempotent by definition; POST usually is not.

## Why it matters
Networks retry. Without idempotency, a retried "charge card" or "place order" can execute twice. It's essential for payments, messaging, and any at-least-once delivery ([[Kafka]]).

## How it works
- Client sends an **idempotency key** (e.g. `Idempotency-Key: <uuid>`) with an unsafe request.
- Server records the key + result. A repeat with the same key returns the stored result instead of re-executing.

```java
@PostMapping("/payments")
public ResponseEntity<PaymentDto> pay(@RequestHeader("Idempotency-Key") String key,
                                      @RequestBody PaymentRequest req) {
    return idempotencyStore.find(key)
        .map(ResponseEntity::ok)
        .orElseGet(() -> {
            PaymentDto result = paymentService.charge(req);
            idempotencyStore.save(key, result);  // atomic insert-if-absent
            return ResponseEntity.ok(result);
        });
}
```

## Production usage
Store keys in [[Redis]] or the database with a TTL. Make the store check-and-set atomic (unique constraint or `SETNX`) to survive concurrent retries. Combine with the [[Kafka|outbox/dedup]] approach for message consumers.

## Trade-offs
- Adds storage and a lookup per request; key scoping/expiry must be designed.

## Common mistakes
- Non-atomic check-then-write (two retries both execute).
- Reusing keys across different request bodies.

## Interview questions
- How would you design an idempotent payment API? (see [[How-would-you-design-an-idempotent-payment-API]])
- How do you prevent duplicate Kafka processing? (see [[How-would-you-prevent-duplicate-Kafka-processing]])

## Related concepts
- [[REST]]
- [[Redis]]
- [[Kafka]]
- [[Retry]]
