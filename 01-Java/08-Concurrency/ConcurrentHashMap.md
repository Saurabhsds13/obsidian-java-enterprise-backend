---
type: concept
domain: java
topic: concurrency
difficulty: hard
status: inbox
tags: [java]
---

# ConcurrentHashMap

## Definition
A thread-safe `Map` designed for high-concurrency reads and writes without locking the entire map.

## Why it matters
It's the correct default for shared mutable maps in multithreaded backends (caches, registries), replacing the fully-synchronized `Hashtable` and `Collections.synchronizedMap`.

## How it works
- Since Java 8 it uses **fine-grained locking at the bucket/node level** (via CAS and `synchronized` on bin heads) rather than the old segment locking.
- Reads are mostly **lock-free** (volatile reads).
- Like [[HashMap]], long bins treeify. Does **not** allow null keys or values (a null would be ambiguous with "absent" under concurrency).
- Aggregate operations (`size`, iterators) are **weakly consistent**: they reflect some state during traversal, not a frozen snapshot.

```java
ConcurrentHashMap<String, AtomicLong> counters = new ConcurrentHashMap<>();
counters.computeIfAbsent(key, k -> new AtomicLong()).incrementAndGet();
```

## Production usage
Use `compute`, `merge`, and `computeIfAbsent` for atomic read-modify-write. Prefer it for in-memory caches and rate-limit counters. For cross-process state use [[Redis]].

## Trade-offs
- Advantages: scalable, non-blocking reads.
- Disadvantages: no null keys/values; compound "check-then-act" across multiple calls is still not atomic — use the atomic methods.

## Common mistakes
- Doing `if (!map.containsKey(k)) map.put(k, v)` (racy) instead of `putIfAbsent`/`computeIfAbsent`.
- Expecting `size()` to be exact under concurrent mutation.

## Interview questions
- How does ConcurrentHashMap achieve thread safety? (see [[synchronized]], [[volatile]])
- Why does it forbid null keys and values?

## Related concepts
- [[HashMap]]
- [[synchronized]]
- [[volatile]]
- [[Java-Memory-Model]]
