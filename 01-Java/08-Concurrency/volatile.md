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
`volatile` is a field modifier that gives a field two guarantees under the [[Java-Memory-Model]]: **visibility** (a write is immediately observable by other threads — reads/writes bypass thread-local caching and hit main memory logically) and **ordering** (a volatile write is a *release* barrier, a volatile read is an *acquire* barrier, so surrounding non-volatile accesses are not reordered across it). It explicitly does **not** provide **atomicity** for compound operations.

## Why it matters
It's the cheapest correct tool for the single-writer/multi-reader "publish a value" and "signal a flag" patterns. Interviewers use it to test whether a candidate understands the difference between *visibility* and *mutual exclusion* — the single most common concurrency misconception.

## How it works — the mechanism
- On the write side the JIT emits a **StoreStore + StoreLoad** barrier (on x86, effectively a locked instruction / `mfence`); on the read side a **LoadLoad + LoadAcquire** barrier. This is why a volatile write "flushes" everything sequenced before it and a volatile read "refreshes" everything after it — release/acquire semantics.
- Reads and writes of a `volatile long`/`double` are **atomic** (this is the one place volatile adds atomicity — it fixes the legacy 64-bit tearing hole). It does *not* make `volatile long x; x++` atomic.
- A volatile read is nearly as cheap as a plain read on x86; the write carries the barrier cost.

## When volatile is correct vs wrong
```java
// CORRECT: single writer, other threads only read the flag
private volatile boolean shuttingDown = false;
public void shutdown() { shuttingDown = true; }
public void loop() { while (!shuttingDown) { pollOnce(); } }

// CORRECT: safe publication of an immutable snapshot (single writer swaps reference)
private volatile PricingTable table;
public void reload(PricingTable fresh) { this.table = fresh; }  // readers see fully-built table

// WRONG: compound read-modify-write is NOT atomic
private volatile int count;
public void hit() { count++; }   // lost updates under concurrency -> use AtomicInteger
```

## The classic use cases
1. **Status flags** (`running`, `shuttingDown`) — one writer, many readers.
2. **Safe publication** of an immutable object by reference swap (config/pricing hot-reload).
3. **Double-checked locking** — the `instance` field must be volatile (see [[Java-Memory-Model]]).
4. **Progress/watermark counters read for monitoring** where exactness isn't required.

## volatile vs AtomicInteger vs synchronized
| Need | Use |
|------|-----|
| See the latest value of a flag/reference | `volatile` |
| Atomic increment / CAS on a single variable | `AtomicInteger`/`AtomicReference` |
| Atomic update across *multiple* fields (invariant) | `synchronized` / `Lock` |

`AtomicInteger` is essentially a volatile int + hardware CAS loop; it gives atomicity *and* visibility. Reach for it the moment you need read-modify-write.

## Trade-offs
- Advantages: cheap, lock-free, no risk of deadlock, precise ordering guarantee.
- Disadvantages: no atomicity, no mutual exclusion; a false sense of safety when multiple volatile fields must change together (each is atomic individually, but the set is not).

## Common mistakes (senior-level)
- `volatile` on a counter expecting atomic increment.
- Guarding a *compound invariant* (e.g. `low <= high`) with two independent volatile fields — you need a lock or an atomic snapshot object.
- Assuming volatile "locks" anything — it never blocks.
- Making a huge object volatile and mutating its internal fields — volatile only governs the *reference*, not the object's fields.

## Interview questions (staff+)
- What are the two guarantees volatile provides and the one it does not?
- Explain release/acquire in terms of barriers; why does a volatile write publish prior plain writes?
- volatile vs AtomicInteger — when is volatile insufficient?
- Why must the DCL `instance` field be volatile?
- Is `volatile long` atomic? Is `volatile long x; x++`?

## Related concepts
- [[Java-Memory-Model]]
- [[synchronized]]
- [[ConcurrentHashMap]]
