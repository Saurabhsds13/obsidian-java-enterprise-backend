---
type: concept
domain: java
topic: concurrency
difficulty: medium
status: inbox
tags: [java]
---

# synchronized

## Definition
A keyword providing mutual exclusion and visibility: only one thread holds a monitor lock at a time, and entering/exiting establishes happens-before ordering.

## Why it matters
It's the simplest correct way to protect shared mutable state and guarantee visibility per the [[Java-Memory-Model]].

## How it works
```java
private final Object lock = new Object();
private int balance;

public void deposit(int amount) {
    synchronized (lock) {   // acquires monitor
        balance += amount;  // atomic w.r.t. other synchronized(lock) blocks
    }                        // release publishes changes (happens-before)
}
```
- Instance methods lock on `this`; static synchronized methods lock on the `Class`.
- Provides both **atomicity** (mutual exclusion) and **visibility** (unlike [[volatile]], which gives visibility only).

## Production usage
Prefer higher-level tools (`ReentrantLock`, `ConcurrentHashMap`, atomics) for contention-heavy paths, but `synchronized` is fine and readable for simple guards.

## Trade-offs
- Advantages: simple, correct, reentrant.
- Disadvantages: coarse locking limits throughput; risk of deadlock with multiple locks.

## Common mistakes
- Locking on a mutable or shared-interned object (e.g. a `String` literal).
- Holding a lock across I/O calls.

## Interview questions
- Difference between `synchronized` and `volatile`?
- What does `synchronized` guarantee besides mutual exclusion?

## Related concepts
- [[volatile]]
- [[Java-Memory-Model]]
- [[ConcurrentHashMap]]
- [[Deadlocks]]
