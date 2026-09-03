---
type: concept
domain: java
topic: collections
difficulty: medium
status: inbox
tags: [java]
---

# Collections Framework

## Definition
The unified architecture of interfaces (`Collection`, `List`, `Set`, `Map`, `Queue`, `Deque`) and implementations for storing and manipulating groups of objects.

## Why it matters
Choosing the right structure is a daily decision that affects correctness and performance.

## How it works
```text
Collection
 ├── List   (ordered, indexed)   -> ArrayList, LinkedList
 ├── Set    (no duplicates)      -> HashSet, LinkedHashSet, TreeSet
 └── Queue/Deque                 -> ArrayDeque, PriorityQueue
Map  (key -> value, not a Collection) -> HashMap, LinkedHashMap, TreeMap
```

## Choosing quickly
- Need index access / order of insertion → [[ArrayList]].
- Need uniqueness → `HashSet` (unordered), `LinkedHashSet` (insertion order), `TreeSet` (sorted).
- Need key→value → [[HashMap]]; sorted keys → `TreeMap`; thread-safe → [[ConcurrentHashMap]].

## Production usage
Prefer interfaces in signatures (`List<T>`, `Map<K,V>`), concrete types at instantiation. Return immutable views (`List.copyOf`) from APIs to prevent external mutation.

## Trade-offs
- Fail-fast iterators throw on structural modification; fail-safe (e.g. `CopyOnWriteArrayList`) don't but cost more.

## Common mistakes
- Using `LinkedList` by default (rarely the right choice).
- Exposing internal mutable collections from getters.

## Interview questions
- fail-fast vs fail-safe iterators?
- When would you pick TreeMap over HashMap?

## Related concepts
- [[ArrayList]]
- [[HashMap]]
- [[ConcurrentHashMap]]
