---
type: concept
domain: java
topic: generics
difficulty: medium
status: inbox
tags: [java]
---

# Generics

## Definition
Parameterized types that let classes and methods operate over types specified at use-site, giving compile-time type safety without casts.

## Why it matters
Generics catch type errors at compile time and make APIs self-documenting. Understanding **type erasure** and **wildcards** explains many "why won't this compile" moments.

## How it works
- **Type erasure**: generic type info is removed at compile time; `List<String>` and `List<Integer>` are both `List` at runtime. This enables backward compatibility but forbids `new T[]` and `instanceof List<String>`.
- **Bounded types**: `<T extends Number>` constrains the type.
- **Wildcards + PECS**: *Producer Extends, Consumer Super*.
  - `? extends T` — read (produce) T.
  - `? super T` — write (consume) T.

```java
// PECS in action
static <T> void copy(List<? super T> dst, List<? extends T> src) {
    for (T item : src) dst.add(item);
}
```

## Production usage
Generic repositories and response wrappers, e.g. `ResponseEntity<ApiResponse<OrderDto>>`. Bounded generics for reusable service utilities.

## Trade-offs
- Advantages: type safety, less casting.
- Disadvantages: erasure limits reflection-based scenarios; wildcards can confuse.

## Common mistakes
- Trying to create generic arrays (`new T[]`).
- Overusing wildcards where a concrete type parameter is clearer.

## Interview questions
- What is type erasure and what are its consequences?
- Explain PECS with an example.

## Related concepts
- [[Collections-Framework]]
- [[Streams]]
