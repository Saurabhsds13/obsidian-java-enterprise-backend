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
An operation is idempotent if applying it multiple times yields the same result/state as applying it once. HTTP GET/PUT/DELETE are idempotent by spec; POST usually isn't. **Idempotency keys** make otherwise-unsafe operations safe to retry.

## Why it matters
Networks retry and clients double-click. Without idempotency, a retried "charge card" or "place order" executes twice. It's mandatory for payments, order creation, and any [[Kafka|at-least-once]] consumer, and it's what makes [[Retry]] safe.

## How it works — the mechanism
1. Client generates a unique **Idempotency-Key** (UUID) per logical operation and sends it as a header.
2. Server, **atomically**, either records the key (first time) and executes, or finds it and returns the **stored result** without re-executing.
3. The atomicity is the crux: a DB **unique constraint** on the key or Redis `SET key NX` ensures two concurrent retries can't both execute.

```java
@PostMapping("/payments")
public ResponseEntity<PaymentDto> pay(@RequestHeader("Idempotency-Key") String key,
                                      @RequestBody PaymentRequest req) {
    // insert-if-absent is the atomic gate (unique index on idempotency_key)
    var existing = idempotency.begin(key, req.fingerprint());
    if (existing.isReplay()) return ResponseEntity.ok(existing.result());   // return stored
    PaymentDto result = paymentService.charge(req);
    idempotency.complete(key, result);
    return ResponseEntity.ok(result);
}
```

## Design details that matter (senior depth)
- **Scope + fingerprint**: bind the key to a request fingerprint (hash of the body) so the *same key with a different body* is rejected (client bug/replay attack), not silently served the old result.
- **In-flight handling**: if a retry arrives while the first is still processing, return `409`/retry-after rather than executing twice (the record is created *before* the side effect).
- **TTL**: keys expire (24–72h typical) — long enough to cover client retries, short enough to bound storage.
- **Storage**: DB unique constraint (durable, transactional with the write) or [[Redis]] (fast, needs care to be atomic with the side effect).

## Idempotency vs the HTTP method table
| Method | Idempotent? | Safe? |
|--------|------------|-------|
| GET | yes | yes (no side effect) |
| PUT | yes (full replace) | no |
| DELETE | yes | no |
| POST | **no** (needs a key) | no |
| PATCH | not necessarily | no |

## Trade-offs
- Adds a storage write + lookup per request; key scoping and expiry must be designed.
- Redis is faster but making the key check atomic *with* the business write is harder than a DB unique constraint in the same transaction.

## Common mistakes (senior-level)
- Non-atomic check-then-write → two retries both execute (the exact bug idempotency should prevent).
- Recording the key *after* the side effect → crash between leaves a gap → duplicate on retry.
- Reusing one key across different request bodies.
- No TTL (unbounded key growth) or TTL shorter than the client's retry window.

## Interview questions (staff+)
- Design an idempotent payment API end to end. ([[How-would-you-design-an-idempotent-payment-API]])
- Why must the key check be atomic, and how do you achieve it?
- How do you handle a retry that arrives while the first request is still in flight?
- How does idempotency make Kafka consumers safe? ([[How-would-you-prevent-duplicate-Kafka-processing]])

## Related concepts
- [[REST]]
- [[Redis]]
- [[Kafka]]
- [[Retry]]
