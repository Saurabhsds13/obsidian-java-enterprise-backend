---
type: concept
domain: dsa
pattern: two-pointers
difficulty: medium
status: inbox
tags: [dsa]
---

# Two Pointers

## Definition
A technique using two indices that move through a data structure (often a sorted array or a list) to solve problems in O(n) time and O(1) space.

## Why it matters
Converts many O(n²) brute-force scans into linear solutions, a staple pattern in interviews.

## When to use
- Sorted array pair/triplet sums.
- Removing duplicates in place.
- Comparing from both ends (palindrome, container with most water).
- Fast/slow variant for cycle detection in linked lists.

## Template
```java
int left = 0, right = arr.length - 1;
while (left < right) {
    int sum = arr[left] + arr[right];
    if (sum == target) return new int[]{left, right};
    if (sum < target) left++;
    else right--;
}
```

## Complexity
- Time: O(n) (after any required sort, O(n log n)).
- Space: O(1).

## Common mistakes
- Forgetting the array must be sorted for the sum variant.
- Off-by-one when both pointers move.

## Interview follow-ups
- Extend two-sum to three-sum (fix one, two-pointer the rest).
- Handle duplicates cleanly.

## Related concepts
- [[Sliding-Window]]
