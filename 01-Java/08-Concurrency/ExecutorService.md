---
type: concept
domain: java
topic: concurrency
difficulty: hard
status: inbox
tags: [java]
---

# ExecutorService

## Definition
An abstraction that decouples **task submission** from **thread management and scheduling**. The workhorse implementation is `ThreadPoolExecutor` (TPE), which manages a pool of worker threads, a work queue, and a rejection policy. `submit()` returns a `Future`; `ScheduledExecutorService` adds delayed/periodic execution.

## Why it matters
Thread-per-request doesn't scale (each platform thread ≈ 1 MB stack + OS scheduling). Pools bound resource usage and, crucially, provide **back-pressure**. The single most common "senior gotcha" is `Executors.newFixedThreadPool` / `newCachedThreadPool` because of their queue/thread unbounded-ness — a real interview and incident topic.

## How it works — TPE lifecycle of a submitted task
```
submit(task)
  ├─ workers < corePoolSize?         -> start a new core thread, run task
  ├─ else: queue.offer(task) succeeds? -> task waits in the work queue
  ├─ else: workers < maxPoolSize?    -> start a new (non-core) thread
  └─ else                            -> RejectedExecutionHandler fires
```
Key knobs: `corePoolSize`, `maximumPoolSize`, `keepAliveTime` (idle non-core reclaim), the **work queue**, and the **rejection policy**.

### The queue choice defines the pool's behavior
| Queue | Effect |
|-------|--------|
| `SynchronousQueue` (no capacity) | hand-off; grows threads to `max`, then rejects (this is `newCachedThreadPool` → unbounded threads) |
| `LinkedBlockingQueue` (unbounded) | queue never fills → `max` is ignored, tasks pile up → **OOM / latent overload** (this is `newFixedThreadPool`) |
| `ArrayBlockingQueue` (bounded) | true back-pressure: fills, spills to extra threads, then rejects — the safe production default |

### Rejection policies
`AbortPolicy` (throw, default), `CallerRunsPolicy` (run on the submitting thread — natural throttle/back-pressure), `DiscardPolicy`, `DiscardOldestPolicy`.

## Enterprise example — a production-safe pool
```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    8, 16,                                   // core, max
    60, TimeUnit.SECONDS,                    // idle reclaim
    new ArrayBlockingQueue<>(1_000),         // bounded -> back-pressure
    new ThreadFactoryBuilder().setNameFormat("orders-%d").build(), // named -> observable
    new ThreadPoolExecutor.CallerRunsPolicy()// throttle producer under overload
);
// graceful shutdown on app stop:
pool.shutdown();
if (!pool.awaitTermination(30, SECONDS)) pool.shutdownNow();
```

## Pool sizing math
- **CPU-bound**: `threads ≈ cores + 1`. More just adds context-switching.
- **I/O-bound (Little's Law)**: `threads ≈ cores × targetUtilization × (1 + waitTime/serviceTime)`. If a task spends 90% waiting on I/O, you need ~10× cores to keep CPUs busy.
- **Virtual threads (Java 21, Project Loom)**: for blocking I/O fan-out, use `Executors.newVirtualThreadPerTaskExecutor()` — millions of cheap JVM-scheduled threads; you no longer size a pool for I/O waiting. **Not** for CPU-bound work, and beware `synchronized` "pinning" a carrier thread (prefer `ReentrantLock` in virtual-thread hot paths on older builds).

## Trade-offs
- Bounded queue + `CallerRuns` protects the service but *slows/rejects* work under overload — that's the point (fail predictably, not catastrophically).
- Unbounded queue gives smooth latency until it silently accumulates a backlog and then OOMs.

## Common mistakes (senior-level)
- `newFixedThreadPool`/`newCachedThreadPool` in production (unbounded queue or unbounded threads).
- Never calling `shutdown()` → threads keep the JVM alive / leak on redeploy.
- Swallowing task failures: exceptions from `submit()` are trapped in the `Future` and only surface on `get()`; with `execute()` they hit the thread's uncaught handler.
- Sharing one pool for CPU-bound and blocking work → blocking starves CPU tasks. Use **separate, isolated pools** (bulkheads).
- Blocking work on the `CompletableFuture` common ForkJoinPool (see [[CompletableFuture]]).

## Interview questions (staff+)
- Walk the exact order TPE uses core threads, the queue, and max threads. Where does the queue choice change everything?
- Why is `newFixedThreadPool` dangerous? What does its queue do to `maximumPoolSize`?
- Size a pool for a task that's 80% I/O wait on an 8-core box.
- What do virtual threads change about pool sizing, and where do they *not* help?
- How do exceptions propagate from `submit` vs `execute`?

## Related concepts
- [[Thread]]
- [[CompletableFuture]]
- [[Circuit-Breaker]]
- [[Java-Memory-Model]]
