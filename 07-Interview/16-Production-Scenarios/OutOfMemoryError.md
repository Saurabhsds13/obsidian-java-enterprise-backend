---
type: production-incident
domain: production
status: inbox
severity: high
tags: [production, java]
---

# OutOfMemoryError

## 1. Symptoms
- `java.lang.OutOfMemoryError: Java heap space`; long GC pauses before the crash; rising memory that never recovers after Full GC.

## 2. Possible causes
- Memory leak (growing live set: caches without eviction, static collections, unclosed resources).
- Undersized heap for real workload; loading huge result sets into memory.
- Metaspace exhaustion (classloader leaks).

## 3. Metrics to inspect
- Heap used after Full GC (trend up = leak), GC frequency/pause, live-set size.

## 4. Logs to inspect
- GC logs (`-Xlog:gc*`), the OOM stack, heap-dump-on-OOM output.

## 5. Traces to inspect
- Requests correlated with memory growth (e.g. an unbounded query/export endpoint).

## 6. Immediate mitigation
- Restart affected instances; roll back a suspect deploy; temporarily raise `-Xmx` to buy time.

## 7. Root cause analysis
- Capture a heap dump (`-XX:+HeapDumpOnOutOfMemoryError` or `jmap`); analyze dominators (MAT) to find the retaining objects.

## 8. Long-term fix
- Fix the leak (bounded caches, close resources, remove static accumulation); stream/paginate large data ([[Pagination]]) instead of loading it all.

## 9. Prevention
- Bounded caches with TTL/eviction; heap dump on OOM enabled; alert on post-GC heap trend; load test memory. See [[Garbage-Collection]].

## Related concepts
- [[Garbage-Collection]]
- [[JVM]]
- [[Pagination]]
