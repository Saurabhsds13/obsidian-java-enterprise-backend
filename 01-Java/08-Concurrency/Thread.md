---
type: concept
domain: java
topic: concurrency
difficulty: medium
status: inbox
tags: [java]
---

# Thread

## Definition
A `Thread` is the unit of concurrent execution. A **platform thread** is a thin wrapper over an OS thread (1:1 mapping, ~1 MB stack, kernel-scheduled). A **virtual thread** (Java 21, Project Loom) is a lightweight thread scheduled by the JVM onto a small pool of platform "carrier" threads, making blocking cheap.

## Why it matters
Everything above it — [[ExecutorService]] pools, [[CompletableFuture]] async, servlet request handling — sits on the threading model. The Loom shift (virtual threads) is a current, high-signal interview topic because it changes how we architect I/O-bound services.

## How it works — the mechanism
- `Runnable` = work returning nothing; `Callable<V>` = work returning `V` and allowed to throw; submit a `Callable` to an executor to get a `Future<V>`.
- `start()` registers the thread with the scheduler and invokes `run()` on the new thread. **Calling `run()` directly executes inline on the current thread** (a classic bug).
- Lifecycle states (`Thread.State`): `NEW → RUNNABLE → (BLOCKED | WAITING | TIMED_WAITING) → TERMINATED`. A thread dump shows these — `BLOCKED` on a monitor points at lock contention; many `WAITING` on a pool queue points at starvation.
- **Interruption** is cooperative: `interrupt()` sets a flag / wakes blocking calls with `InterruptedException`. Code must check `isInterrupted()` or handle the exception — never swallow it silently (restore with `Thread.currentThread().interrupt()`).

### Platform vs virtual threads
| | Platform thread | Virtual thread |
|--|-----------------|----------------|
| Backed by | OS thread (1:1) | JVM-scheduled onto carriers (M:N) |
| Cost | ~1 MB stack, limited to ~thousands | ~few hundred bytes, millions feasible |
| Best for | CPU-bound work | blocking I/O fan-out (one thread per request/task) |
| Blocking | ties up an OS thread | unmounts the carrier — cheap |
| Pooling | pool them | **don't pool**; one per task |

## Enterprise example
```java
// Modern I/O-bound style: a virtual thread per task, no pool sizing
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<Result>> futures = ids.stream()
        .map(id -> executor.submit(() -> callDownstream(id)))  // blocking is fine here
        .toList();
    for (var f : futures) collect(f.get());
}
```

## Trade-offs
- Platform threads: predictable, ideal for compute; scarce and expensive for high-concurrency blocking.
- Virtual threads: massive concurrency for blocking I/O; **not** faster for CPU-bound work; watch for `synchronized` **pinning** (holds the carrier — prefer `ReentrantLock` in hot paths on early Loom builds) and unbounded concurrency hitting downstreams (still need [[Rate-Limiting]]/semaphores).

## Common mistakes (senior-level)
- Calling `run()` instead of `start()`.
- Creating raw threads per request (unbounded) — use an executor or virtual threads.
- Swallowing `InterruptedException` (breaks cancellation/shutdown); always restore the flag or propagate.
- Pooling virtual threads (defeats their purpose).
- Assuming virtual threads speed up CPU-bound code.

## Interview questions (staff+)
- `Runnable` vs `Callable`; `start()` vs `run()`.
- Explain the thread states and what a thread dump full of `BLOCKED` vs `WAITING` tells you.
- What are virtual threads, when do they help, and what is carrier pinning?
- How does cooperative interruption work, and why must you restore the interrupt flag?
- Why don't you pool virtual threads?

## Related concepts
- [[ExecutorService]]
- [[CompletableFuture]]
- [[synchronized]]
- [[Java-Memory-Model]]
