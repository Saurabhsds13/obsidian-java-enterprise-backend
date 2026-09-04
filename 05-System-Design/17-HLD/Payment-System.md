---
type: system-design
level: HLD
domain: system-design
difficulty: hard
status: inbox
tags: [system-design]
---

# Payment System

## 1. Problem
Process payments correctly — never double-charge, never lose money, always be auditable — while integrating with external payment providers that can time out with unknown results.

## 2. Requirements
### Functional
- Initiate charge, capture, refund; handle provider webhooks; immutable audit/ledger.
### Non-functional
- **Correctness above all**; strong consistency for money state; full auditability; high availability; idempotent under client + network retries.

### Capacity estimation
```
1,000 payments/s peak; each = a few short DB transactions + 1 provider call (~200-800ms)
Provider latency dominates -> use async/non-blocking to the provider, DO NOT hold DB tx across it
Ledger: 1000/s × 86400 ≈ 86M entries/day -> partition by account/time; archive cold data
Idempotency keys retained 24-72h in a fast store + unique constraint in DB
```

## 3. API design
```text
POST /api/v1/payments   (header: Idempotency-Key)  -> {paymentId, status}
POST /webhooks/provider  (verify HMAC signature)    -> update payment status
```

## 4. Data model
`payments(id, idempotency_key UNIQUE, amount, currency, status, provider_ref, version)`,
`ledger_entries(...)` double-entry, append-only.

## 5. High-level architecture
API → payment service (short DB tx: persist INITIATED + idempotency key) → **async** provider call → webhook/callback updates status → **outbox** publishes domain events. See [[Message-Queues]] outbox pattern.

## 6. Component responsibilities
- **Payment service**: enforce the **state machine** INITIATED → AUTHORIZED → CAPTURED / FAILED / REFUNDED; reject illegal transitions.
- **Outbox relay**: publish events reliably after commit ([[Kafka]]).
- **Reconciliation job**: resolve unknown/timeout outcomes by querying the provider.

## 7. Database choice
Relational, [[ACID]], strong consistency ([[CAP-Theorem|CP]] side) with a double-entry ledger as the source of truth.

## 8. Cache strategy
Minimal on money paths (correctness over latency); cache only read-only reference data.

## 9. Messaging strategy
**Transactional outbox**: write payment state + outbox event in one DB transaction, relay to [[Kafka]] async — avoids the dual-write problem (committed DB but lost publish). Downstream consumers are idempotent.

## 10. Scaling strategy
Partition by account; keep transactions short ([[Database-Transactions]]); provider call is async so DB connections aren't held during the slow external hop ([[Connection-Pooling]]).

## 11. Failure scenarios
**Provider timeout with unknown result** (the hard case): never blindly retry a charge → the idempotency key + a **status query / webhook reconciliation** determines the true outcome. Network retries from the client hit the same idempotency key → return the stored result.

## 12. Consistency decisions
Strong consistency for balances/state; [[Idempotency]] key + DB unique constraint prevents double charge; state machine forbids illegal transitions; events are eventually consistent for downstream.

## 13. Security considerations
Verify webhook HMAC signatures (reject spoofed callbacks), minimize PCI scope (tokenize card data via the provider), secrets in a manager (`YOUR_API_KEY`), least-privilege access to the ledger.

## 14. Observability
Authorization/decline rates, provider latency + error rate, reconciliation backlog (unknown-state count), duplicate-key hits ([[Observability]]); alert on any ledger imbalance.

## 15. Bottlenecks
Provider latency/limits; DB contention on hot accounts (partition + short tx).

## 16. Trade-offs
Correctness/consistency chosen over availability and latency on the money path; async provider calls + outbox add eventual consistency for downstream in exchange for not holding transactions open.

## 17. Interview discussion points
Idempotent charge design ([[How-would-you-design-an-idempotent-payment-API]]), outbox vs dual-write, handling the unknown-provider-result timeout, refund/chargeback flows, exactly-once vs effectively-once for downstream consumers.

## Related concepts
- [[Idempotency]]
- [[ACID]]
- [[Message-Queues]]
- [[CAP-Theorem]]
- [[Kafka]]
