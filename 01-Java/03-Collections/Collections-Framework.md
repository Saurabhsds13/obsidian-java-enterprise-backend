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
The unified set of interfaces (`Collection`, `List`, `Set`, `Queue`, `Deque`, `Map`) and implementations for storing and manipulating groups of objects, plus algorithms (`Collections`, `Arrays`) and iteration contracts.

## Why it matters
Picking the right structure is a daily decision that determines algorithmic complexity and, at scale, the difference between an endpoint that's fast and one that isn't. Interviewers expect instant, justified choices and knowledge of iterator semantics.

## How it works — the hierarchy and complexity
```
Iterable
 └ Collection
    ├ List   (ordered, indexed, duplicates)  -> ArrayList, LinkedList
    ├ Set    (no duplicates)                 -> HashSet, LinkedHashSet, TreeSet
    └ Queue/Deque                            -> ArrayDeque, PriorityQueue, LinkedList
Map (not a Collection)                        -> HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap
```
| Structure | get/contains | add | remove | ordering |
|-----------|-------------|-----|--------|----------|
| ArrayList | O(1) index | amortized O(1) end | O(n) | insertion |
| LinkedList | O(n) | O(1) at ends | O(1) at ends | insertion |
| HashSet/HashMap | O(1) avg | O(1) avg | O(1) avg | none |
| TreeSet/TreeMap | O(log n) | O(log n) | O(log n) | sorted |
| ArrayDeque | — | O(1) ends | O(1) ends | insertion |
| PriorityQueue | O(1) peek | O(log n) | O(log n) | heap order |

## fail-fast vs fail-safe iterators
- **fail-fast** (`ArrayList`, `HashMap`): track a `modCount`; structural modification during iteration throws `ConcurrentModificationException` (a best-effort bug detector, *not* a concurrency guarantee — even single-threaded `for-each` + `remove` triggers it).
- **fail-safe / weakly consistent** (`CopyOnWriteArrayList`, [[ConcurrentHashMap]]): iterate over a snapshot or tolerate concurrent change; no CME, but may not see latest writes.
- Correct in-loop removal: `Iterator.remove()` or `Collection.removeIf(...)`.

## Enterprise example — interface-typed APIs, immutable returns
```java
public interface OrderRepository {
    List<Order> findRecent(int limit);          // expose the interface, not ArrayList
}
// Return an unmodifiable view so callers can't mutate internal state:
return List.copyOf(internalList);               // Java 10+; also Map.copyOf, Set.copyOf
```

## Choosing quickly (interview reflex)
- Index access / iteration → `ArrayList` (rarely `LinkedList`).
- Uniqueness → `HashSet`; + insertion order → `LinkedHashSet`; + sorted → `TreeSet`.
- Key→value → `HashMap`; sorted keys/range → `TreeMap`; thread-safe → `ConcurrentHashMap`.
- Stack/queue/deque → `ArrayDeque` (faster than `Stack`/`LinkedList`).
- Priority scheduling → `PriorityQueue` (binary heap).

## Trade-offs
- `Comparable` (natural order, one ordering) vs `Comparator` (external, many orderings). `TreeMap`/`TreeSet` require one or the other and reject `null` keys (unlike `HashMap`).

## Common mistakes (senior-level)
- Defaulting to `LinkedList` (poor cache locality; `ArrayList`/`ArrayDeque` almost always win).
- Removing inside a `for-each` loop (CME) instead of `removeIf`/iterator.
- Returning internal mutable collections from getters (encapsulation leak).
- Using `Stack`/`Vector` (legacy, synchronized) instead of `ArrayDeque`.
- Assuming `CopyOnWriteArrayList` is cheap for writes (it copies the whole array per write — reads-heavy only).

## Interview questions (staff+)
- fail-fast vs fail-safe iterators — is CME a concurrency guarantee?
- When would you choose `TreeMap` over `HashMap`?
- `Comparable` vs `Comparator`; how do `TreeSet` and `null` interact?
- Why is `ArrayDeque` preferred over `Stack` and `LinkedList`?
- When is `CopyOnWriteArrayList` the right choice?

## Related concepts
- [[ArrayList]]
- [[HashMap]]
- [[ConcurrentHashMap]]
