---
type: concept
domain: java
topic: functional
difficulty: hard
status: inbox
tags: [java]
---

# Streams

## Definition
A pipeline abstraction (Java 8+) for declarative, composable processing of element sequences. A stream is **not** a data structure — it carries no storage; it describes a computation over a source that is executed lazily when a terminal operation runs.

## Why it matters
Streams make transformations readable and enable easy parallelism, but the laziness, statefulness, and parallel semantics are exactly where correctness/performance bugs hide — making them a rich senior interview topic.

## How it works — the mechanism
- A pipeline = **source** (collection, array, generator) → **intermediate ops** (lazy: `map`, `filter`, `sorted`) → **terminal op** (eager: `collect`, `reduce`, `forEach`, `count`).
- **Laziness + fusion**: intermediate ops build a chain of `Sink`s; nothing runs until the terminal op. Elements are pushed through the whole chain **one at a time** (not stage-by-stage over the whole dataset), so `filter().map().findFirst()` short-circuits without materializing intermediates.
- **Spliterator** is the traversal + splitting primitive; for parallel streams it recursively splits the source and the common ForkJoinPool runs segments, then results are combined.
- **Stateful vs stateless** ops: `map`/`filter` are stateless; `sorted`/`distinct`/`limit` are stateful (may buffer or need the whole input), which limits parallel efficiency.

## Enterprise example — collectors do the real work
```java
Map<Currency, BigDecimal> settledByCcy = payments.stream()
    .filter(p -> p.status() == SETTLED)                 // lazy
    .collect(Collectors.groupingBy(
        Payment::currency,
        Collectors.reducing(BigDecimal.ZERO, Payment::amount, BigDecimal::add)));

// Teeing (Java 12+): compute two aggregates in one pass
var stats = payments.stream().collect(Collectors.teeing(
    Collectors.counting(),
    Collectors.summingDouble(p -> p.amount().doubleValue()),
    (count, sum) -> new Stats(count, sum)));
```

## Parallel streams — when they help and hurt
`parallelStream()` uses the shared common ForkJoinPool. It helps only when: large N, cheap-to-split source (arrays, `ArrayList` — not `LinkedList`/`IntStream.iterate`), CPU-bound stateless work, and the combine step is cheap. It **hurts** when: small collections, I/O or blocking work (starves the shared pool used by everything else), stateful/ordered ops, or shared mutable state (data race). Rule: measure; and never do blocking I/O in a parallel stream.

## Trade-offs
- Advantages: declarative, composable, lazy/short-circuiting, easy `groupingBy`/`partitioningBy`.
- Disadvantages: harder to debug/step than loops; parallelism is a footgun; a stream is single-use; not always faster than a plain loop for simple cases.

## Common mistakes (senior-level)
- **Side effects** in `map`/`filter` (should be pure) — breaks under reordering/parallelism.
- Reusing a consumed stream → `IllegalStateException`.
- `parallelStream()` for I/O-bound work or small data (starves common pool, adds overhead).
- Mutating shared state in `forEach` instead of using a `Collector`.
- Using `peek` for logic (it's for debugging and may be skipped when the pipeline optimizes).

## Interview questions (staff+)
- Intermediate vs terminal, and what "lazy" precisely means (per-element fusion + short-circuit).
- How does a parallel stream execute (spliterator split → common FJP → combine)? When is it a bad idea?
- Stateless vs stateful intermediate ops and their effect on parallelism.
- Why must lambdas passed to streams be side-effect-free?
- `reduce` vs `collect` — mutable vs immutable reduction.

## Related concepts
- [[Optional]]
- [[Generics]]
- [[CompletableFuture]]
