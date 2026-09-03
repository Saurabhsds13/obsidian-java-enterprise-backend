---
type: adr
status: accepted
decision: Use PostgreSQL as the primary relational database
date: 2026-09-03
tags: [adr, database]
---

# ADR-0001: Why PostgreSQL

## Status
accepted

## Context
The system needs a primary datastore for transactional, relational data (orders, payments, users) requiring [[ACID]] guarantees, rich querying, and strong ecosystem support. Team familiarity and operational maturity matter.

## Decision
Use **PostgreSQL** as the primary relational database.

## Alternatives
- **MySQL** — solid and widely used, but Postgres offers richer SQL (window functions, CTEs, JSONB), stronger default correctness, and extensibility.
- **A NoSQL store (e.g. MongoDB)** — flexible schema and easy horizontal scaling, but weaker multi-row transactional guarantees and relational querying for this domain.
- **A managed cloud-proprietary DB** — convenient, but increases lock-in.

## Consequences
- Strong transactional correctness for money/state; powerful querying and indexing ([[Indexes]], [[EXPLAIN]]).
- Clear scaling path: read replicas ([[Replication]]) first, [[Sharding]] only if writes exceed one primary.
- Team must manage Postgres-specific tuning (vacuum, connection limits — see [[Connection-Pooling]]).

## Trade-offs
- Relational rigor over schema flexibility; single-primary write scaling until sharded. Accepted because correctness and query power outweigh these for this domain.

## Related concepts
- [[ACID]]
- [[Replication]]
- [[Connection-Pooling]]
