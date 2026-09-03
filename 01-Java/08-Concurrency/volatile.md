---
type: concept
domain: java
topic: concurrency
difficulty: medium
status: inbox
tags: [java]
---

# volatile

## Definition
A field modifier guaranteeing **visibility** and ordering: reads/writes go to main memory and are not reordered around, establishing happens-before between a write and subsequent reads.

## Why it matters
It solves the "one thread's write is never seen by another" problem cheaply, without the mutual exclusion cost of [[synchronized]].

## How it works
```java
private volatile boolean running = true;

public void stop() { running = false; }        // visible to all threads immediately
public void loop() { while (running) { /* work */ } }
```
- Guarantees **visibility** and prevents certain reorderings.
- Does **not** provide atomicity for compound actions (`count++` is read-modify-write and still races). Use `AtomicInteger` for that.

## Production usage
Flags (`running`, `shuttingDown`), and the classic double-checked-locking singleton (`volatile` instance field).

## Trade-offs
- Advantages: cheap visibility.
- Disadvantages: no atomicity, easy to misuse for counters.

## Common mistakes
- Using `volatile` for `count++` expecting atomicity.

## Interview questions
- What does `volatile` guarantee, and what does it NOT?
- Why is `volatile` needed in double-checked locking?

## Related concepts
- [[synchronized]]
- [[Java-Memory-Model]]
