---
type: concept
domain: database
topic: connection-pooling
difficulty: medium
status: inbox
tags: [database]
---

# Connection Pooling

## Definition
Reusing a fixed set of pre-established database connections instead of opening a new one per request.

## Why it matters
Connections are expensive to create and databases cap them. A pool bounds usage, cuts latency, and provides back-pressure. Pool exhaustion is a classic production incident.

## How it works
- A pool (e.g. **HikariCP**, Spring Boot's default) keeps N open connections; requests borrow and return them.
- Key settings: `maximum-pool-size`, `connection-timeout` (wait for a free connection), `max-lifetime`, `idle-timeout`.
- Pool size should roughly match the DB's capacity and the app's real concurrency — bigger is not better.

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      connection-timeout: 2000   # ms to wait for a connection
```

## Production usage
Size from measurement: often `pool = cores * factor`, tuned to DB `max_connections` across all app instances. Ensure connections are released promptly — long [[Database-Transactions]] hold them.

## Trade-offs
- Too small → threads wait/time out; too large → DB overload and context-switching.

## Common mistakes
- Oversized pools overwhelming the DB.
- Holding connections during long transactions or remote calls → **pool exhaustion**.

## Interview questions
- Why can a bigger pool hurt?
- How would you debug connection pool exhaustion? (see [[Connection-Pool-Exhausted]])

## Related concepts
- [[Database-Transactions]]
- [[Query-Optimization]]
- [[Replication]]
