---
type: concept
domain: java
topic: collections
difficulty: easy
status: inbox
tags: [java]
---

# ArrayList

## Definition
A resizable-array implementation of `List`. Backed by a contiguous array that grows automatically.

## Why it matters
It's the default list. Understanding its growth and access characteristics prevents accidental O(n) behavior in hot paths.

## How it works
- Random access `get(i)`/`set(i)` is O(1).
- `add` at the end is amortized O(1); when full, the backing array grows (~1.5x) and elements are copied.
- Insert/remove in the middle is O(n) (shifting).

```java
List<Order> orders = new ArrayList<>(expectedSize); // pre-size to avoid regrowth
orders.add(order);
Order first = orders.get(0);
```

## Production usage
Pre-size when the count is known. For heavy queue-like add/remove at the front, use `ArrayDeque` or `LinkedList` instead.

## Trade-offs
- Advantages: cache-friendly, fast random access.
- Disadvantages: O(n) middle insert/remove; resizing copies.

## Common mistakes
- Removing elements inside a for-each loop (throws `ConcurrentModificationException`) — use an `Iterator` or `removeIf`.

## Interview questions
- ArrayList vs LinkedList — when to use each?
- How does ArrayList grow?

## Related concepts
- [[Collections-Framework]]
- [[HashMap]]
