---
type: concept
domain: dsa
pattern: sliding-window
difficulty: medium
status: inbox
tags: [dsa]
---

# Sliding Window

## Definition
Maintaining a moving sub-range (window) over a sequence, expanding and shrinking it to satisfy a constraint, to avoid recomputing from scratch.

## Why it matters
Turns many O(n·k) or O(n²) substring/subarray problems into O(n).

## When to use
- Longest/shortest substring or subarray satisfying a condition.
- Max/min sum of a fixed-size window.
- Counting subarrays with a property.

## Template (variable window)
```java
int left = 0, best = 0;
Map<Character,Integer> count = new HashMap<>();
for (int right = 0; right < s.length(); right++) {
    count.merge(s.charAt(right), 1, Integer::sum);
    while (/* window invalid */ count.size() > k) {
        char c = s.charAt(left++);
        if (count.merge(c, -1, Integer::sum) == 0) count.remove(c);
    }
    best = Math.max(best, right - left + 1);
}
return best;
```

## Complexity
- Time: O(n) (each element enters/leaves the window once).
- Space: O(k) for the window state.

## Common mistakes
- Shrinking the window incorrectly (invariant not restored).
- Fixed vs variable window confusion.

## Interview follow-ups
- Fixed-size vs variable-size windows.
- Track window state with a hashmap vs frequency array.

## Related concepts
- [[Two-Pointers]]
