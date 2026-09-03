---
type: concept
domain: java
topic: functional
difficulty: medium
status: inbox
tags: [java]
---

# Streams

## Definition
The Stream API (Java 8+) processes sequences of elements with declarative, composable operations (`map`, `filter`, `reduce`, `collect`).

## Why it matters
Streams make data transformations readable and enable parallelism, but misuse causes performance and correctness surprises.

## How it works
- A pipeline has a **source**, zero or more **intermediate** ops (lazy: `map`, `filter`), and one **terminal** op (eager: `collect`, `reduce`, `forEach`).
- Nothing runs until the terminal op; intermediate ops are fused into a single pass.

```java
Map<Currency, BigDecimal> totals = payments.stream()
    .filter(p -> p.status() == SETTLED)
    .collect(Collectors.groupingBy(
        Payment::currency,
        Collectors.reducing(BigDecimal.ZERO, Payment::amount, BigDecimal::add)));
```

## Production usage
Great for in-memory transformations and aggregations. For large data, keep operations stateless and side-effect-free. Prefer `Collectors.groupingBy`/`partitioningBy` over manual maps.

## Trade-offs
- Advantages: readable, composable, lazy.
- Disadvantages: harder to debug than loops; `parallelStream` helps only for CPU-bound work on large datasets and can hurt with shared state or small collections.

## Common mistakes
- Side effects inside `map`/`filter`.
- Reusing a consumed stream (throws `IllegalStateException`).
- Reaching for `parallelStream` without measuring.

## Interview questions
- Intermediate vs terminal operations, and laziness?
- When do parallel streams help or hurt?

## Related concepts
- [[Optional]]
- [[Generics]]
