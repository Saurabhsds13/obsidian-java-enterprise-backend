---
type: production-incident
domain: production
status: inbox
severity: high
tags: [production, database]
---

# Database Connection Pool Exhausted

## 1. Symptoms
- Requests hang then fail with "unable to acquire connection" / connection-timeout errors.
- Latency spikes; thread pool fills; health checks may fail.

## 2. Possible causes
- Connections held too long (long [[Database-Transactions]], remote calls inside transactions).
- Pool too small for real concurrency, or a connection leak (not returned).
- Downstream DB slowness making every query hold a connection longer.

## 3. Metrics to inspect
- HikariCP active/idle/pending connections, acquire time.
- DB active sessions vs `max_connections`.
- Query latency, transaction duration.

## 4. Logs to inspect
- Connection-timeout stack traces; slow-query log; leak-detection warnings.

## 5. Traces to inspect
- Spans where a DB connection is held across an external call.

## 6. Immediate mitigation
- Restart to clear leaked connections (temporary); reduce inbound load / shed traffic.
- Kill long-running queries/transactions on the DB.

## 7. Root cause analysis
- Find code holding connections during I/O or long transactions; check for missing `close()`/leaks; verify pool sizing vs DB capacity.

## 8. Long-term fix
- Move remote calls out of transactions; keep transactions short; fix leaks (try-with-resources / Spring-managed).
- Right-size the pool across all instances vs DB `max_connections`; add [[Query-Optimization]].

## 9. Prevention
- Enable leak detection; alert on pending connections and acquire time; load test.

## Related concepts
- [[Connection-Pooling]]
- [[Database-Transactions]]
- [[Query-Optimization]]
