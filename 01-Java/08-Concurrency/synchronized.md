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
`synchronized` acquires an object's intrinsic **monitor lock**, providing three things under the [[Java-Memory-Model]]: **mutual exclusion** (one thread in the critical section per monitor), **visibility** (unlock happens-before subsequent lock, so changes are published), and **ordering** (the lock/unlock act as full barriers). It is **reentrant** — the owning thread can re-acquire the same monitor without deadlocking.

## Why it matters
It's the baseline correctness primitive and the one most people use without understanding its *visibility* guarantee. Architect-level questions probe lock granularity, the difference from `ReentrantLock`, and why holding a lock across I/O is a production incident waiting to happen.

## How it works — the mechanism
- Bytecode: a synchronized block compiles to `monitorenter`/`monitorexit`; a synchronized method sets the `ACC_SYNCHRONIZED` flag and the JVM does the enter/exit implicitly (with exit on both normal and exceptional return).
- **What object is locked?**
  - instance method → `this`
  - static method → the `Class` object
  - block → the explicit object expression
- **JVM lock optimizations**: historically biased locking (removed/disabled by default in modern JDKs), *thin/lightweight* locks using CAS on the object header's mark word for the uncontended case, inflating to a heavyweight OS monitor (thread parking) only under contention. Uncontended `synchronized` is therefore cheap; contention is what costs.

## Enterprise example — correct lock scoping
```java
public class AccountService {
    private final Map<Long, Object> locks = new ConcurrentHashMap<>();

    // Per-key locking: high concurrency across different accounts,
    // exclusion only between operations on the SAME account.
    public void transfer(long from, long to, BigDecimal amount) {
        // canonical lock ordering prevents deadlock (always low id first)
        Object first  = lockFor(Math.min(from, to));
        Object second = lockFor(Math.max(from, to));
        synchronized (first) {
            synchronized (second) {
                debit(from, amount);
                credit(to, amount);
            }
        }
    }
    private Object lockFor(long id) { return locks.computeIfAbsent(id, k -> new Object()); }
}
```
Two lessons: (1) **fine-grained keys** beat one global lock for throughput; (2) **consistent lock ordering** is how you prevent the classic AB/BA deadlock ([[Deadlocks]]).

## synchronized vs ReentrantLock
| Feature | `synchronized` | `ReentrantLock` |
|---------|---------------|-----------------|
| Acquire/release | implicit, block-scoped | explicit `lock()`/`unlock()` (use `finally`) |
| Try / timeout | no | `tryLock()`, `tryLock(timeout)` |
| Interruptible acquire | no | `lockInterruptibly()` |
| Fairness | no (unfair) | optional fair mode |
| Condition queues | one (`wait`/`notify`) | multiple `Condition`s |
| Readability / safety | harder to leak | must remember `unlock()` in finally |

Rule of thumb: prefer `synchronized` for simple guards (JVM optimizes it, can't forget to release); reach for `ReentrantLock` when you need timeout, interruptibility, fairness, or multiple condition queues. For read-heavy state use `ReadWriteLock`/`StampedLock`; for a single variable prefer `Atomic*`.

## Trade-offs
- Advantages: simple, reentrant, correct visibility, JVM-optimized uncontended path, auto-release on exception.
- Disadvantages: coarse locking serializes throughput; no timeout means a stuck lock holder blocks everyone; deadlock risk with multiple monitors.

## Common mistakes (senior-level)
- **Holding a lock across a remote/DB call** — one slow downstream stalls every thread contending the monitor → cascading failure. Compute outside, lock only the state mutation.
- Locking on a mutable field or an interned `String`/`Integer` cache (another component may lock the same interned instance).
- Locking on `this` in a public class, letting callers accidentally (or maliciously) lock the same monitor — prefer a `private final Object lock`.
- Inconsistent lock ordering across code paths → deadlock.
- Using `synchronized` where an `Atomic*` or immutable snapshot + `volatile` would be lock-free and faster.

## Interview questions (staff+)
- What three guarantees does `synchronized` provide? (mutual exclusion is the *famous* one; visibility is the one people forget)
- Object monitor semantics: what's locked for instance vs static methods?
- `synchronized` vs `ReentrantLock` — when do you actually need the Lock?
- How do biased/thin/heavyweight locks affect the cost of uncontended vs contended synchronized?
- Why is holding a monitor across I/O dangerous, and how do you refactor it?

## Related concepts
- [[Java-Memory-Model]]
- [[volatile]]
- [[ConcurrentHashMap]]
- [[Deadlocks]]
