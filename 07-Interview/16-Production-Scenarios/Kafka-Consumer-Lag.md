---
type: production-incident
domain: production
status: inbox
severity: medium
tags: [production, backend]
---

# Kafka Consumer Lag

## 1. Symptoms
- Consumer lag (offset behind log end) grows; downstream data is stale; processing falls behind producers.

## 2. Possible causes
- Consumers too few for partition count, or slow per-message processing.
- A poison message stalling a partition; downstream dependency slow.
- Rebalancing storms from long processing between polls.

## 3. Metrics to inspect
- Consumer lag per partition, records-consumed-rate, processing time, rebalance frequency.

## 4. Logs to inspect
- Consumer errors/retries, rebalance events, dead-letter routing.

## 5. Traces to inspect
- Per-message processing spans; slow downstream calls ([[Timeout]]).

## 6. Immediate mitigation
- Scale consumers up to the partition count; skip/route poison messages to a dead-letter topic; increase throughput temporarily.

## 7. Root cause analysis
- Is it throughput (need more consumers/partitions) or per-message slowness (optimize handler / downstream)?

## 8. Long-term fix
- Right-size partitions and consumers; make processing fast and idempotent ([[Idempotency]]); add DLQ; tune `max.poll.records`/`max.poll.interval.ms`.

## 9. Prevention
- Alert on lag thresholds; load test consumer throughput; keep handlers non-blocking.

## Related concepts
- [[Kafka]]
- [[Idempotency]]
- [[Timeout]]
