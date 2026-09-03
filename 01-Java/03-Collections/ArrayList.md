---
type: concept
domain: java
topic: collections
difficulty: medium
status: inbox
tags: [java]
---

# ArrayList

## Definition
A resizable-array implementation of `List`. Backed by a contiguous `Object[]` that grows automatically, giving O(1) random access and amortized O(1) append, at the cost of O(n) insert/remove in the middle.

## Why it matters
It's the default list and the right answer in the overwhelming majority of cases. Understanding its growth policy and cache behavior is what lets you justify it over `LinkedList` and avoid accidental O(n) hot paths.

## How it works — the mechanism
- Backed by an array + a `size` field. `get(i)`/`set(i)` are O(1) bounds-checked array accesses.
- **Growth**: when full, it allocates a new array of **~1.5× capacity** (`oldCap + (oldCap >> 1)`) and `System.arraycopy`s the elements. Each individual grow is O(n), but *amortized* append is O(1) because growth is geometric.
- **Insert/remove at index i**: shifts `size - i` elements → O(n).
- **Cache locality**: contiguous storage is CPU-cache-friendly, so even O(n) scans are fast in practice — a big reason it beats `LinkedList` for iteration.

## Enterprise example
```java
// Pre-size when the count is known to avoid repeated grow+copy:
List<OrderLine> lines = new ArrayList<>(order.expectedLineCount());
lines.addAll(fetched);

// Safe in-loop removal:
lines.removeIf(l -> l.quantity() == 0);   // not: for-each + remove -> CME
```

## ArrayList vs LinkedList (the perennial question)
| Operation | ArrayList | LinkedList |
|-----------|-----------|------------|
| get/set by index | O(1) | O(n) |
| append (end) | amortized O(1) | O(1) |
| insert/remove at ends | O(n)/O(1) | O(1) |
| insert/remove middle | O(n) (shift) | O(n) to *find* + O(1) to relink |
| memory | compact | per-node object + 2 pointers |
| cache locality | excellent | poor |

Verdict: prefer `ArrayList` almost always. For queue/stack/deque semantics use `ArrayDeque`, not `LinkedList`. `LinkedList` only wins for frequent add/remove at *both ends while holding an iterator* — rare.

## Trade-offs
- Advantages: fast random access, cache-friendly, low overhead.
- Disadvantages: O(n) middle insert/remove; growth copies; not thread-safe.

## Common mistakes (senior-level)
- Reaching for `LinkedList` by reflex.
- `for-each` + `remove` → `ConcurrentModificationException`; use `removeIf`/iterator.
- Repeated `add` without pre-sizing on a large known count (many grow+copy cycles).
- `list.remove(int)` vs `list.remove(Object)` ambiguity with `Integer` elements (index vs value).
- Sharing across threads without synchronization (use `CopyOnWriteArrayList` for read-heavy, or external locking).

## Interview questions (staff+)
- Growth factor and why append is amortized O(1)?
- ArrayList vs LinkedList — give a case where LinkedList actually wins.
- Why does cache locality make ArrayList scans fast despite "O(n)"?
- The `remove(int)` vs `remove(Object)` gotcha with `Integer`.

## Related concepts
- [[Collections-Framework]]
- [[HashMap]]
