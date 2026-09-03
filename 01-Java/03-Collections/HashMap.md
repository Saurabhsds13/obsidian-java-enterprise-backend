---
type: concept
domain: java
topic: collections
difficulty: hard
status: inbox
tags: [java]
---

# HashMap

## Definition
A hash-table-based `Map` giving average O(1) `get`/`put` by mapping a key's hash to a bucket in a backing array. It permits one null key and null values, preserves no order, and is **not thread-safe**. Since Java 8, individual buckets convert from linked lists to red-black trees under heavy collision to bound worst-case lookup.

## Why it matters
It's the most-used data structure in backend Java and the deepest "do you actually understand it" interview probe — it touches hashing, the [[equals-and-hashCode]] contract, resizing/rehashing, treeification, and thread-safety (the reason [[ConcurrentHashMap]] exists).

## How it works — the mechanism
- **Backing array** of `Node<K,V>[]` buckets, default **capacity 16**, always a power of two (so index = `hash & (n-1)`, a cheap bitmask instead of modulo).
- **Hash spread**: `HashMap` doesn't use `key.hashCode()` directly. It applies `h ^ (h >>> 16)` to mix high bits into low bits — otherwise keys whose hashes differ only in high bits would collide in a small table.
- **put(k,v)**:
  1. compute spread hash → index
  2. empty bucket → place node
  3. collision → walk the bucket comparing `hash` then `equals()`; replace on match, else append
  4. bucket reaches **TREEIFY_THRESHOLD (8)** *and* table ≥ **MIN_TREEIFY_CAPACITY (64)** → convert list to red-black tree (O(log n)); below 64 it resizes instead
- **get(k)**: spread hash → bucket → compare `hash`+`equals()` → return match.

### Resize / rehash
When `size > capacity × loadFactor` (default **0.75**), capacity **doubles** and entries are redistributed. In Java 8 the split is clever: an entry either stays at index `i` or moves to `i + oldCap`, decided by one bit — no full rehash of every key. Resizing is O(n) and allocates a new array; pre-size to avoid repeated resizes.

## Enterprise example — pre-sizing and safe keys
```java
// Pre-size to avoid resize churn when the count is known (initialCapacity is a hint):
Map<String, Account> accounts = new HashMap<>((int) (expected / 0.75f) + 1);

// Keys must be effectively immutable in the fields used by equals/hashCode:
record CacheKey(String tenant, long id) {}   // record => correct equals/hashCode, immutable
Map<CacheKey, Value> cache = new HashMap<>();
```

## Why load factor is 0.75
It's the empirical sweet spot: lower (e.g. 0.5) wastes memory but reduces collisions; higher (e.g. 1.0) packs tighter but lengthens buckets. 0.75 balances space vs collision probability (Poisson distribution of bucket sizes makes long chains rare at 0.75).

## Thread-safety failure
A plain `HashMap` under concurrent writes can corrupt structure; pre-Java 8, a concurrent resize could form a **cycle in a bucket's linked list → infinite loop (100% CPU)**. Java 8 removed the infinite-loop shape but concurrent mutation is still undefined. Use [[ConcurrentHashMap]] for shared maps.

## HashMap vs LinkedHashMap vs TreeMap
| | HashMap | LinkedHashMap | TreeMap |
|--|---------|---------------|---------|
| Order | none | insertion (or access) order | sorted by key |
| Lookup | O(1) avg | O(1) avg | O(log n) |
| Backed by | array + bins | array + doubly-linked list | red-black tree |
| Use | default | LRU caches (access-order), stable iteration | range/sorted queries |

## Trade-offs
- Advantages: fast average case, flexible, null-friendly.
- Disadvantages: no ordering, not thread-safe, sensitive to poor `hashCode()` (degrades to O(log n) tree or O(n) if also non-`Comparable`).

## Common mistakes (senior-level)
- Sharing a plain `HashMap` across threads.
- Mutable keys whose hash changes after insertion → entry becomes unreachable.
- Overriding `equals` but not `hashCode` (see [[equals-and-hashCode]]) → lost lookups.
- Assuming iteration order is stable (it isn't — use `LinkedHashMap`).
- Forgetting `initialCapacity` is pre-load-factor, so pass `count/0.75 + 1`.

## Interview questions (staff+)
- Walk `put` end to end, including hash spread, collision handling, and treeification thresholds (8 / 64).
- How does Java 8 resize avoid rehashing every key?
- Why load factor 0.75, and why power-of-two capacity?
- What exactly went wrong with concurrent `HashMap` pre-Java 8?
- Why must keys be effectively immutable?

## Related concepts
- [[equals-and-hashCode]]
- [[ConcurrentHashMap]]
- [[Collections-Framework]]
- [[How-does-HashMap-work-internally]]
