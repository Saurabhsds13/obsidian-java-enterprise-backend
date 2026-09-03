---
type: concept
domain: java
topic: concurrency
difficulty: hard
status: inbox
tags: [java]
---

# Java Memory Model

## Definition
The JMM (JSR-133, formalized in Java 5) is the part of the language spec that defines **when a write by one thread is guaranteed visible to a read by another**, and **which reorderings** the compiler, JIT, and CPU are allowed to perform. Its central abstraction is the **happens-before** partial order. If there is no happens-before edge between a write and a read of the same non-final field, the read may observe the write, a stale value, or (for non-atomic 64-bit values pre-Java 17 semantics) a torn value — the behavior is simply *undefined* for a data race.

## Why it matters
Every concurrency primitive ([[synchronized]], [[volatile]], `Lock`, `Atomic*`, [[ConcurrentHashMap]]) is ultimately *defined in terms of the JMM*. Interviewers probe it because it separates people who memorize "add volatile" from those who can reason about why a specific field needs it and what ordering guarantee they're buying. Getting it wrong produces bugs that pass every test on x86 and fail intermittently on ARM (weaker memory model) in production.

## How it works — the mechanism

### The three problems the JMM addresses
1. **Visibility** — a value written to a CPU core's store buffer / L1 cache may never reach another core without a memory barrier.
2. **Ordering** — compilers and CPUs reorder independent instructions for performance; single-threaded semantics (as-if-serial) are preserved, but other threads can observe the reordering.
3. **Atomicity** — `long`/`double` writes were permitted to tear pre-JMM; references and ints are atomic, but compound ops (`i++`) never are.

### Happens-before edges (the ones worth memorizing)
- **Program order**: within one thread, each action happens-before the next in source order.
- **Monitor lock**: unlock of monitor M happens-before every subsequent lock of M ([[synchronized]]).
- **Volatile**: a write to a volatile field happens-before every subsequent read of that field ([[volatile]]).
- **Thread start**: `Thread.start()` happens-before any action in the started thread.
- **Thread termination**: all actions in a thread happen-before another thread returns from `join()` on it.
- **Transitivity**: if A hb B and B hb C, then A hb C. This is the key that makes volatile publish *non-volatile* data.

### The canonical publication pattern
```java
class FlagPublish {
    private int data;                 // plain field
    private volatile boolean ready;   // volatile "gate"

    void writer() {
        data = 42;        // (1) plain write
        ready = true;     // (2) volatile write — releases everything before it
    }
    int reader() {
        if (ready) {      // (3) volatile read — acquires
            return data;  // (4) guaranteed to see 42
        }
        return -1;
    }
}
```
`(1) hb (2)` by program order, `(2) hb (3)` by the volatile rule, `(3) hb (4)` by program order → transitively `(1) hb (4)`. This is **release/acquire** semantics: the volatile write is a *release* barrier, the volatile read is an *acquire* barrier.

### Safe publication (the interview trap)
An object is *safely published* only if the reference is shared through: a `final` field (set in the constructor), a volatile/AtomicReference, a value guarded by a lock, or via a `static` initializer, or a concurrent collection. Publishing via a plain field means readers may see a **partially constructed object** (reference visible before the constructor's field writes).

- **`final` fields** get a special JMM freeze: after a constructor returns normally, any thread that reads the object through a properly-published reference is guaranteed to see the correctly-initialized final fields — *without* extra synchronization. This is why immutable objects are cheap to share and why leaking `this` from a constructor breaks the guarantee.

## Enterprise example — lazy singleton done correctly
```java
public final class RateLimiterRegistry {
    private static volatile RateLimiterRegistry instance;   // volatile is load-bearing

    private final Map<String, Bucket> buckets;              // final -> safe publication
    private RateLimiterRegistry() { this.buckets = new ConcurrentHashMap<>(); }

    public static RateLimiterRegistry getInstance() {
        RateLimiterRegistry local = instance;               // read volatile once (perf)
        if (local == null) {
            synchronized (RateLimiterRegistry.class) {
                local = instance;
                if (local == null) {
                    instance = local = new RateLimiterRegistry();
                }
            }
        }
        return local;
    }
}
```
Without `volatile`, double-checked locking is broken: another thread can see a non-null `instance` whose constructor writes (the map) are not yet visible → NPE or a half-built map. Modern advice: prefer the **initialization-on-demand holder** idiom (a static nested class) which uses classloader guarantees and needs no volatile.

## Trade-offs / cost model
| Construct | Guarantees | Approx cost | Use when |
|-----------|-----------|-------------|----------|
| plain field | none across threads | free | thread-confined data |
| `final` field | safe publication of value | free after construction | immutable objects |
| `volatile` | visibility + ordering, **no atomicity** | cheap (barrier, no lock) | flags, single-writer publish |
| `synchronized`/`Lock` | visibility + ordering + **mutual exclusion** | uncontended cheap (biased/thin), contended expensive (park) | compound invariants |
| `Atomic*` (CAS) | atomic RMW, visibility | cheap under low contention, spins under high | counters, lock-free structures |

## Common mistakes (senior-level)
- Assuming x86's strong (TSO) memory model means missing `volatile` is "fine" — it fails on ARM/POWER and the compiler can still reorder.
- Using `volatile` for `count++` (read-modify-write is not atomic — needs `AtomicInteger` or a lock).
- Leaking `this` from a constructor (registering a listener, starting a thread) → other threads see an unfrozen object.
- Believing `synchronized` only provides mutual exclusion — it *also* provides the visibility barrier, which is why guarded plain fields are safe.
- Double-checked locking without `volatile`.

## Interview questions (staff+)
- Define happens-before and walk the edges that make a volatile flag safely publish a plain field.
- Why is DCL broken without volatile? What exactly can go wrong?
- What does `final` guarantee under the JMM and how does leaking `this` defeat it?
- Release/acquire vs sequential consistency — what does volatile actually give you?
- Why does the same code behave differently on x86 vs ARM?

## Related concepts
- [[volatile]]
- [[synchronized]]
- [[ConcurrentHashMap]]
- [[Thread]]
