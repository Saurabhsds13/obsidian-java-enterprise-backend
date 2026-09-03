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
Automatic reclamation of heap memory occupied by objects no longer reachable from GC roots (stack references, statics, JNI).

## Why it matters
GC pauses and throughput directly affect service latency. Tuning and diagnosing GC is a senior backend skill.

## How it works
- **Reachability**: objects reachable from roots are live; the rest are collectible.
- **Generational hypothesis**: most objects die young. The heap splits into **Young** (Eden + Survivor) and **Old** generations.
  - **Minor GC** collects Young (frequent, cheap).
  - **Major/Full GC** touches Old (rarer, more expensive).
- Collectors:
  - **G1** (default since Java 9): region-based, targets pause-time goals.
  - **ZGC / Shenandoah**: low-pause, large-heap concurrent collectors.

```bash
java -XX:+UseG1GC -Xms2g -Xmx2g -XX:MaxGCPauseMillis=200 -Xlog:gc* App
```

## Production usage
Watch GC logs and metrics for pause frequency/duration. Rising Old-gen occupancy after Full GC signals a leak. Set `-Xms == -Xmx` in containers to avoid resize pauses.

## Trade-offs
- Throughput vs pause time: no collector wins both; pick by SLO.

## Common mistakes
- Calling `System.gc()` in application code.
- Blaming GC for what is actually a memory leak (growing live set).

## Interview questions
- Minor vs Major vs Full GC?
- How would you diagnose rising memory / an OutOfMemoryError?

## Related concepts
- [[JVM]]
- [[Java-Memory-Model]]
