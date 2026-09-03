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
`HashMap` is a hash-table-based `Map` storing key–value pairs with average O(1) `get`/`put`. It permits one null key and null values and is **not** thread-safe.

## Why it matters
It's the most-used collection in backend code and a perennial interview topic because its internals touch hashing, [[equals-and-hashCode]], resizing, and (since Java 8) treeification.

## How it works
- Backed by an array of **buckets**. A key's bucket index is derived from `hash(key)` spread over the array length.
- Collisions (multiple keys in one bucket) form a **linked list**; since Java 8, a bucket with ≥ 8 entries converts to a **red-black tree** (O(log n) worst case) when the table is large enough.
- **Load factor** (default 0.75) and **capacity** (default 16) govern resizing. When `size > capacity * loadFactor`, the table doubles and entries are **rehashed**.

```java
Map<String, Account> accounts = new HashMap<>();
accounts.put("acc-1", account);   // hash("acc-1") -> bucket
Account a = accounts.get("acc-1"); // recompute hash -> same bucket -> equals() match
```

## Why equals/hashCode matter here
Lookup finds the bucket via `hashCode()`, then walks it comparing with `equals()`. A broken contract (see [[equals-and-hashCode]]) means entries get "lost."

## Production usage
Pre-size when the count is known: `new HashMap<>(expected / 0.75f + 1)` to avoid repeated resizing. For concurrent access use [[ConcurrentHashMap]] — a shared plain `HashMap` under concurrent writes can corrupt or (pre-Java 8) infinite-loop.

## Trade-offs
- Advantages: fast average-case, flexible.
- Disadvantages: no ordering, not thread-safe, sensitive to poor hash functions.

## Common mistakes
- Sharing a `HashMap` across threads without synchronization.
- Using mutable keys whose hash changes after insertion.

## Interview questions
- How does HashMap work internally? (see [[How-does-HashMap-work-internally]])
- What changed in Java 8 (treeification)? What is the load factor?

## Related concepts
- [[equals-and-hashCode]]
- [[ConcurrentHashMap]]
- [[Collections-Framework]]
- [[ArrayList]]
