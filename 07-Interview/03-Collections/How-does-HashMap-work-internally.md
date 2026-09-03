---
type: interview
domain: java
topic: collections
difficulty: medium
status: inbox
tags: [interview, java]
---

# How does HashMap work internally?

## Question
> Explain how `HashMap` stores and retrieves entries internally.

## Short answer
It's an array of buckets; a key's hash selects a bucket, collisions form a linked list (or a red-black tree since Java 8 when a bucket gets large), and resizing rehashes entries when the load factor is exceeded.

## Detailed answer
`put(k,v)` computes `hash(k)`, spreads it, and maps it to a bucket index. If the bucket has entries, it walks them comparing with `equals()`; a matching key is updated, otherwise the entry is appended. Since Java 8, a bucket with ≥ 8 entries (in a table ≥ 64) converts to a tree for O(log n) worst case. When `size > capacity * loadFactor` (default 16 × 0.75), capacity doubles and entries are rehashed. `get(k)` recomputes the hash, finds the bucket, and returns the `equals()` match.

## Example
```java
Map<String, Integer> m = new HashMap<>();
m.put("a", 1);   // hash("a") -> bucket
m.get("a");      // same bucket -> equals -> 1
```

## Production relevance
Pre-size to avoid resizing; never share a plain `HashMap` across threads (use [[ConcurrentHashMap]]).

## Common mistake
Assuming O(1) always — a bad `hashCode()` degrades to O(n)/O(log n). Breaking the [[equals-and-hashCode]] contract loses entries.

## Follow-up questions
- What changed in Java 8? What is the load factor?
- Why must keys be effectively immutable?

## Related concepts
- [[HashMap]]
- [[equals-and-hashCode]]
- [[ConcurrentHashMap]]
