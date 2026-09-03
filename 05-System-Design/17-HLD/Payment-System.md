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
Process payments correctly and exactly once, integrating with external payment providers.

## 2. Requirements
### Functional
- Initiate a charge, capture, refund; handle provider callbacks/webhooks; maintain an audit trail.
### Non-functional
- Correctness above all (no double charge, no lost money), auditability, high availability, strong consistency for money state.
### Scale assumptions
- Thousands of TPS with strict correctness.

## 3. API design
```text
POST /api/v1/payments  (Idempotency-Key required) -> { paymentId, status }
POST /webhooks/provider  (verify signature)
```

## 4. Data model
`payments(id, idempotency_key UNIQUE, amount, currency, status, provider_ref, version)`, `ledger(entries...)`.

## 5. High-level architecture
API → payment service (DB transaction) → provider adapter; webhooks update status; outbox publishes events.

## 6. Component responsibilities
- Payment service: state machine (INITIATED → AUTHORIZED → CAPTURED / FAILED / REFUNDED).
- Outbox worker: reliably publish events after commit.

## 7. Database choice
Relational, [[ACID]], with a double-entry ledger; strong consistency ([[CAP-Theorem|CP]] side).

## 8. Cache strategy
Minimal for money paths; cache read-only reference data only.

## 9. Messaging strategy
**Outbox pattern**: write event + state in one transaction, publish to [[Kafka]] async — avoids lost events and dual-write inconsistency.

## 10. Scaling strategy
Partition by account; keep transactions short ([[Database-Transactions]]).

## 11. Failure scenarios
Timeout to provider with unknown result → reconcile via status query/webhook; retries guarded by [[Idempotency]].

## 12. Consistency decisions
Strong consistency for balances; [[Idempotency]] keys prevent double charge; state machine forbids illegal transitions.

## 13. Security considerations
Verify webhook signatures, PCI scope minimization, secrets in a manager (`YOUR_API_KEY`), least privilege.

## 14. Observability
Success/decline rates, provider latency, reconciliation gaps, alerting ([[Observability]]).

## 15. Bottlenecks
Provider latency/limits; DB contention on hot accounts.

## 16. Trade-offs
Correctness/consistency over availability; async events add eventual consistency for downstream consumers.

## 17. Interview discussion points
Idempotent charge design, outbox vs dual-write, handling unknown provider results, refund/chargeback flows.

## Related concepts
- [[Idempotency]]
- [[ACID]]
- [[Kafka]]
- [[CAP-Theorem]]
