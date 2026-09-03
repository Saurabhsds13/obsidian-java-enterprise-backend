---
type: concept
domain: java
topic: jvm
difficulty: hard
status: inbox
tags: [java]
---

# Garbage Collection

## Definition
Automatic reclamation of heap memory holding objects no longer **reachable** from GC roots (thread stacks, statics, JNI, active registers). Java GCs are tracing collectors; modern ones are generational and/or region-based, trading throughput, latency (pause time), and footprint.

## Why it matters
GC pause frequency and duration directly shape service tail latency (p99/p999). Choosing and tuning a collector, and distinguishing a leak from an undersized heap, are core senior/architect skills and frequent incident root causes.

## How it works — the mechanism

### Reachability + the generational hypothesis
- **Reachability**: mark from roots; unreachable objects are collectible. (Reference-counting isn't used — it can't collect cycles.)
- **Generational hypothesis**: *most objects die young*. So the heap splits into **Young** (Eden + two Survivor spaces) and **Old**:
  - **Minor GC**: collects Young; survivors are copied Eden→Survivor→(after enough tenuring age)→Old. Frequent, cheap, uses copying (compacts for free).
  - **Major/Full GC**: touches Old; rarer, more expensive.
- **Write barriers + remembered sets/card tables** track Old→Young references so a Minor GC needn't scan the whole Old gen.

### Collectors (pick by SLO)
| Collector | Model | Optimizes | Typical use |
|-----------|-------|-----------|-------------|
| **Serial** | single-thread | footprint | tiny heaps / single core |
| **Parallel** | multi-thread STW | **throughput** | batch jobs |
| **G1** (default 9+) | region-based, incremental | balanced, pause-target | most services |
| **ZGC / Shenandoah** | mostly-concurrent, region | **low pause** (sub-ms, huge heaps) | latency-critical, large heaps |

- **G1** divides the heap into equal regions, collects the regions with the most garbage first ("garbage first"), and targets `MaxGCPauseMillis`.
- **ZGC/Shenandoah** do marking and relocation *concurrently* with the app (using load/read barriers), keeping pauses ~sub-millisecond largely independent of heap size — at some throughput/CPU cost.

## Enterprise example — sizing + observability
```bash
java -XX:+UseG1GC -Xms4g -Xmx4g \      # Xms == Xmx: avoid heap resize pauses in containers
     -XX:MaxGCPauseMillis=200 \
     -Xlog:gc*:file=/logs/gc.log:time,uptime:filecount=5,filesize=10m \
     -jar app.jar
```
Set `-Xms == -Xmx` in containers so the heap doesn't grow/shrink at runtime.

## Diagnosis: leak vs undersized heap
- Plot **heap used immediately after each Full GC**. Flat/sawtooth → healthy. **Monotonically rising** → memory leak (growing live set) → heap dump + MAT dominator tree to find the retaining root (unbounded cache, static collection, thread-local, unclosed resource).
- Frequent Full GCs with high reclaim → heap too small → raise `-Xmx` (verify it's not a leak first).
- Long single pauses → wrong collector for the SLO → move to G1/ZGC.

## Trade-offs
- **The GC trilemma**: you can favor throughput, pause time, or footprint — not all three. Parallel wins throughput; ZGC wins latency; Serial wins footprint. Choose by SLO.

## Common mistakes (senior-level)
- Calling `System.gc()` in app code (can trigger a Full GC / stall).
- Blaming GC for what is a **leak** (rising live set), then just raising `-Xmx` (delays the OOM, doesn't fix it).
- Ignoring **allocation rate** — excessive short-lived garbage drives Minor GC frequency; fix by reducing allocation, not just tuning.
- Using the throughput collector on a latency-sensitive service.
- Not setting `-Xms == -Xmx` in containers (resize pauses).

## Interview questions (staff+)
- Explain the generational hypothesis and Minor vs Major vs Full GC.
- How does G1 differ from Parallel, and when do you reach for ZGC/Shenandoah?
- How do you distinguish a memory leak from an undersized heap from GC logs/dumps?
- What are write barriers / card tables for?
- The throughput vs latency vs footprint trade-off — how does it drive collector choice?

## Related concepts
- [[JVM]]
- [[Java-Memory-Model]]
- [[OutOfMemoryError]]
