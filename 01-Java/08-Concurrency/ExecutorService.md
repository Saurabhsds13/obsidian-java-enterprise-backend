---
type: concept
domain: java
topic: concurrency
difficulty: medium
status: inbox
tags: [java]
---

# ExecutorService

## Definition
An abstraction for asynchronously executing tasks on a managed pool of threads, decoupling task submission from thread lifecycle.

## Why it matters
Creating threads per task doesn't scale. Pools bound resource usage and enable back-pressure, which is essential in production services.

## How it works
```java
ExecutorService pool = new ThreadPoolExecutor(
    4, 8, 60, TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(1000),          // bounded queue = back-pressure
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);

Future<Report> f = pool.submit(() -> buildReport());
Report report = f.get();   // blocks until done
pool.shutdown();
```
Key params: core/max pool size, keep-alive, work queue, and **rejected execution handler**.

## Production usage
- Use **bounded** queues; an unbounded queue hides overload until OOM.
- Size CPU-bound pools ~= cores; I/O-bound pools larger (or use virtual threads).
- Always `shutdown()` on app stop. Name threads for observability.
- Prefer `Executors.newFixedThreadPool` cautiously (unbounded queue) — an explicit `ThreadPoolExecutor` is safer.

## Trade-offs
- Bounded queue + sensible rejection policy protects the service but can drop/slow work under overload (by design).

## Common mistakes
- Unbounded queues masking overload.
- Not handling `Future` exceptions (swallowed until `get()`).

## Interview questions
- How do you size a thread pool?
- What happens when the queue is full?

## Related concepts
- [[Thread]]
- [[CompletableFuture]]
- [[Circuit-Breaker]]
