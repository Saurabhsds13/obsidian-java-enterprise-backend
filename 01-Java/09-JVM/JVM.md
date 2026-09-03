---
type: concept
domain: java
topic: jvm
difficulty: hard
status: inbox
tags: [java]
---

# JVM

## Definition
The Java Virtual Machine is a stack-based abstract machine that loads, verifies, and executes **bytecode**, providing platform independence, automatic memory management ([[Garbage-Collection]]), and adaptive optimization (JIT). HotSpot is the reference implementation.

## Why it matters
The JVM is where performance, memory footprint, startup, and GC behavior are actually decided. Architect-level candidates are expected to reason about runtime data areas, class loading, JIT tiers, and how to diagnose a misbehaving process — not just write Java.

## How it works — the mechanism

### Compilation + execution pipeline
```
.java --javac--> .class (bytecode) --ClassLoader--> [verify -> link -> init]
   --> Interpreter (fast start) --profiles hot methods--> JIT (C1/C2) --> native code
```
- Bytecode runs on the **operand stack** (JVM is stack-based, not register-based).
- **Interpretation first** for fast startup; the JIT compiles *hot* methods to native code guided by runtime profiles (**tiered compilation**: C1 quick/lightly-optimized → C2 aggressive). C2 does inlining, escape analysis (stack allocation / lock elision), loop unrolling, and can **deoptimize** back to the interpreter when a speculative assumption breaks.

### Runtime data areas
| Area | Scope | Holds | OOM type |
|------|-------|-------|----------|
| **Heap** | shared | all objects, arrays | `OutOfMemoryError: Java heap space` |
| **Metaspace** | shared | class metadata (off-heap, native since Java 8; replaced PermGen) | `OutOfMemoryError: Metaspace` |
| **JVM Stack** | per thread | frames: locals, operand stack | `StackOverflowError` |
| **PC register** | per thread | current instruction | — |
| **Native method stack** | per thread | JNI frames | — |
| **Code cache** | shared | JIT-compiled native code | code cache full → deopt |

### Class loading (delegation model)
- Loaders: Bootstrap → Platform → Application, with **parent-first delegation** (a loader asks its parent before loading itself) — prevents core classes being overridden and enables isolation.
- Phases: **loading → linking (verify, prepare, resolve) → initialization** (static init, lazy on first active use).

## Enterprise example — container-aware runtime flags
```bash
java -XX:+UseG1GC \
     -XX:MaxRAMPercentage=75 \        # size heap relative to the container memory limit
     -XX:MaxMetaspaceSize=256m \
     -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/dumps \
     -Xlog:gc*:file=/logs/gc.log \
     -jar app.jar
```
Modern JVMs are container-aware (respect cgroup limits), but pinning `MaxRAMPercentage` and enabling heap-dump-on-OOM are production hygiene.

## Diagnosis toolkit (know these names)
- `jcmd <pid> GC.heap_info`, thread dump `jstack`/`jcmd Thread.print`, heap dump `jmap`/`GC.heap_dump`, live profiling **JFR** (`-XX:StartFlightRecording`) + Mission Control, async-profiler for CPU/alloc flame graphs.
- **Heap OOM with rising post-GC live set** = leak → heap dump + Eclipse MAT dominator tree.
- **Metaspace OOM** = classloader leak (common with hot redeploys / dynamic proxies).

## Trade-offs
- Advantages: portability, world-class GC + JIT, deep observability.
- Disadvantages: JIT **warmup** (cold code is interpreted — matters for latency SLOs and short-lived functions), memory overhead vs native. Mitigations: tiered compilation, AppCDS (class-data sharing), or GraalVM native-image for fast startup at the cost of peak throughput/JIT.

## Common mistakes (senior-level)
- Assuming `-Xmx` bounds total process memory (Metaspace, thread stacks ~1 MB each, direct byte buffers, and JIT code cache live outside the heap).
- Benchmarking cold (interpreted) code and reporting it as steady-state (need warmup / JMH).
- Ignoring container limits pre-Java 10 (JVM saw host memory, not cgroup limit → OOM-killed).
- Confusing `StackOverflowError` (deep recursion) with heap OOM.

## Interview questions (staff+)
- Enumerate the runtime data areas and which OOM each produces.
- What does the JIT do; explain tiered compilation and deoptimization.
- Walk class loading phases and parent-first delegation — why does delegation matter?
- Heap vs Metaspace vs stack — what lives where, and how do you size them in a container?
- How would you diagnose a memory leak vs an undersized heap?

## Related concepts
- [[JDK-vs-JRE]]
- [[Garbage-Collection]]
- [[Java-Memory-Model]]
- [[OutOfMemoryError]]
