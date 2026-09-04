---
type: concept
domain: database
topic: connection-pooling
difficulty: hard
status: inbox
tags: [database]
---

# Connection Pooling

## Definition
Reusing a bounded set of pre-established database connections instead of opening one per request. A TCP connect + auth + backend process/thread setup is expensive; a pool (HikariCP is Spring Boot's default) amortizes that, bounds concurrency, and provides back-pressure.

## Why it matters
Databases cap connections (each Postgres connection is a backend process ~5–10 MB; each MySQL connection a thread). Pool exhaustion — requests hanging waiting for a free connection — is a classic production incident, and correct sizing is counterintuitive (bigger is usually worse).

## How it works — the mechanism
- The pool keeps N open connections; a request **borrows** one, uses it for a query/transaction, and **returns** it. If none is free within `connectionTimeout`, it fails fast.
- Key settings (HikariCP):
  - `maximumPoolSize` — the ceiling (the number that matters most).
  - `connectionTimeout` — max wait for a connection (fail fast, e.g. 2–5 s).
  - `maxLifetime` — recycle connections before DB/LB idle-kills them.
  - `leakDetectionThreshold` — warn when a connection is held too long (finds leaks).

### Why smaller pools are often faster (the counterintuitive part)
A database has limited CPU/disk parallelism. Past a point, more concurrent connections just cause **context-switching, lock contention, and cache thrashing** — throughput *drops* and latency rises. A common heuristic: `connections ≈ (core_count × 2) + effective_spindle_count`. For many services the right per-instance pool is **single digits to low tens**, not 100. Crucially, size the pool against the DB's total capacity **across all app instances**: 20 instances × 50 = 1000 connections can exceed the DB's `max_connections`.

## Enterprise example
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10          # per instance; × instances must stay < DB max_connections
      minimum-idle: 10               # keep it steady (avoid churn)
      connection-timeout: 3000       # ms; fail fast rather than pile up
      max-lifetime: 1740000          # < DB/LB idle timeout (e.g. 29 min for a 30 min timeout)
      leak-detection-threshold: 20000
```

## Trade-offs
- Too small → threads wait, `connectionTimeout` errors under load.
- Too large → DB overload (context switching, memory, lock contention), and can exceed `max_connections`.
- For very high fan-out, a **server-side pooler** (PgBouncer in transaction mode) multiplexes thousands of app connections onto few DB connections.

## Common mistakes (senior-level)
- Oversizing the pool ("more = faster") → overwhelms the DB.
- Ignoring the ×instances multiplication vs DB `max_connections`.
- Holding a connection across a long transaction or a remote call ([[Database-Transactions]]) → **exhaustion** (see [[Connection-Pool-Exhausted]]).
- Connection leaks (not returned) → slow drain to zero available.
- `maxLifetime` ≥ DB/LB idle timeout → borrowing a silently-dead connection.

## Interview questions (staff+)
- Why can a smaller pool outperform a larger one?
- How do you size a pool against DB `max_connections` with many instances?
- How would you debug connection pool exhaustion? ([[Connection-Pool-Exhausted]])
- What does PgBouncer transaction-mode pooling add?
- Why must `maxLifetime` be below the DB/LB idle timeout?

## Related concepts
- [[Database-Transactions]]
- [[Query-Optimization]]
- [[Replication]]
- [[Connection-Pool-Exhausted]]
