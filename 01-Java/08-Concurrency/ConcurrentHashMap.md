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
A hash-based `ConcurrentMap` engineered for concurrent access: reads are (almost always) lock-free, writes lock only the single bin being modified, and iterators are **weakly consistent** (they never throw `ConcurrentModificationException` and reflect *some* state during traversal, not a frozen snapshot). It forbids `null` keys and values.

## Why it matters
It is the default shared mutable map in every real backend (in-memory caches, registries, rate-limit counters). Interviewers use it to test understanding of lock striping vs bin-level locking, the atomic compound methods, and why "check-then-act" across two calls is still a race.

## How it works — the mechanism (Java 8+)
- Same bucket/table structure as [[HashMap]]: array of bins, bins are linked lists that **treeify** to red-black trees at ≥ 8 entries (with table ≥ 64), giving O(log n) worst case.
- **Concurrency is no longer segment-based** (pre-Java 8 used ~16 `Segment` locks). Now:
  - **Empty bin insert**: a **CAS** on the bin head — fully lock-free, no monitor.
  - **Non-empty bin update**: `synchronized` on the **bin head node only** — so contention is per-bin, and N bins give ~N-way write parallelism.
  - **Reads**: traverse via `volatile` reads of nodes/table — lock-free, see a consistent-enough view.
- **Resizing is concurrent/cooperative**: multiple threads help transfer bins; a `ForwardingNode` marks migrated bins so readers/writers follow to the new table.
- **`size()`** is maintained via a striped counter (`CounterCell[]`, the LongAdder technique) to avoid a single hot contended counter; hence it's an estimate under concurrent mutation.

### Why no null keys/values
Under concurrency, `map.get(k) == null` is ambiguous between "absent" and "present with null value" — and there's no atomic way to disambiguate without locking, so nulls are banned by design (unlike [[HashMap]]).

## Enterprise example — atomic compound operations
```java
// Rate-limit counters: atomic get-or-create + increment, no external lock
ConcurrentHashMap<String, LongAdder> hits = new ConcurrentHashMap<>();
hits.computeIfAbsent(userId, k -> new LongAdder()).increment();

// Cache with atomic single-flight load (only ONE thread computes per key)
ConcurrentHashMap<Key, Value> cache = new ConcurrentHashMap<>();
Value v = cache.computeIfAbsent(key, this::expensiveLoad);
```
Caveat: the mapping function in `computeIfAbsent`/`compute` runs **while holding the bin lock**, so it must be **short and must not** update the *same* map (can deadlock/livelock) or do blocking I/O under contention.

## The race the compound methods fix
| Racy (two calls) | Atomic (one call) |
|------------------|-------------------|
| `if (!m.containsKey(k)) m.put(k,v)` | `m.putIfAbsent(k, v)` |
| `if (m.containsKey(k)) m.get(k)...put` | `m.compute(k, ...)` / `m.merge(k, ...)` |
| read counter, `+1`, write back | `m.merge(k, 1, Integer::sum)` |

## ConcurrentHashMap vs alternatives
| Option | Concurrency | Notes |
|--------|-------------|-------|
| `ConcurrentHashMap` | bin-level lock + lock-free reads | default choice |
| `Collections.synchronizedMap` | one global lock | serializes everything; iteration needs external sync |
| `Hashtable` | one global lock | legacy, avoid |
| `HashMap` (shared) | **unsafe** | corruption / (pre-8) infinite loop on resize |
| `ConcurrentSkipListMap` | lock-free, **sorted** | when you need ordering + concurrency |

## Common mistakes (senior-level)
- Compound "check-then-act" across two separate calls instead of `computeIfAbsent`/`merge`.
- Doing blocking I/O or updating the same map inside `compute*` (runs under the bin lock).
- Treating `size()`/iteration as an exact snapshot.
- Sharing a plain [[HashMap]] "because it works in tests" — corruption is timing-dependent.
- Using it as a distributed cache — it's single-JVM; cross-process needs [[Redis]].

## Interview questions (staff+)
- How does CHM achieve thread safety in Java 8+ (CAS on empty bin, synchronized bin head)? How did it change from segments?
- Why are null keys/values forbidden?
- What runs under the lock in `computeIfAbsent`, and what must you avoid there?
- Why is `size()` only an estimate, and how is the counter implemented (LongAdder striping)?
- How does concurrent resize stay correct (ForwardingNode, cooperative transfer)?

## Related concepts
- [[HashMap]]
- [[Java-Memory-Model]]
- [[synchronized]]
- [[volatile]]
