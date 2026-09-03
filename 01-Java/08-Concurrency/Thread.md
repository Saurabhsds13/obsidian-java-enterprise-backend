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
A `Thread` is the unit of concurrent execution in Java, mapping (in the classic model) to an OS thread. `Runnable`/`Callable` define the work.

## Why it matters
Concurrency underpins throughput in backend services. Understanding threads underlies pools ([[ExecutorService]]), async ([[CompletableFuture]]), and the [[Java-Memory-Model]].

## How it works
```java
Runnable task = () -> log.info("running on {}", Thread.currentThread().getName());
Thread t = new Thread(task, "worker-1");
t.start();       // start() spawns a new thread; run() would execute inline
```
- `Callable<V>` returns a value and can throw; run via an executor to get a `Future<V>`.
- Threads are expensive (memory for stack, OS scheduling), so real code uses pools, not raw threads.
- **Virtual threads** (Java 21) provide cheap, JVM-scheduled threads for high-concurrency blocking I/O.

## Production usage
Never create raw threads per request. Use [[ExecutorService]] pools sized to the workload, or virtual threads for I/O-bound fan-out.

## Trade-offs
- Platform threads: limited count, higher cost.
- Virtual threads: cheap for blocking I/O, not for CPU-bound work.

## Common mistakes
- Calling `run()` instead of `start()`.
- Creating unbounded threads and exhausting memory.

## Interview questions
- Difference between `Runnable` and `Callable`?
- What are virtual threads and when do they help?

## Related concepts
- [[ExecutorService]]
- [[CompletableFuture]]
- [[synchronized]]
- [[Java-Memory-Model]]
