---
type: concept
domain: java
topic: generics
difficulty: hard
status: inbox
tags: [java]
---

# Generics

## Definition
Parameterized types that give compile-time type safety and eliminate casts, implemented via **type erasure**: the compiler checks generic types, then erases them to raw types (or their bounds) so the bytecode is compatible with pre-generics code. Generics are a *compile-time* feature with almost no runtime footprint.

## Why it matters
Erasure explains a whole class of "why won't this compile / why does this warn" questions (no `new T[]`, no `instanceof List<String>`, unchecked warnings) and the wildcard/PECS rules are a standard senior interview filter.

## How it works — the mechanism
- **Erasure**: `List<String>` and `List<Integer>` are both just `List` at runtime; a type parameter `T` erases to `Object` (or its leftmost bound, e.g. `<T extends Number>` erases to `Number`). The compiler inserts **synthetic casts** at call sites and generates **bridge methods** to preserve polymorphism after erasure.
- **Consequences of erasure**:
  - Cannot do `new T[]` or `new T()` (type unknown at runtime).
  - Cannot use `instanceof List<String>` (only `instanceof List<?>`).
  - Cannot have two overloads that differ only by generic type (same erasure).
  - Static fields can't use the class type parameter (shared across all parameterizations).
- **Bounded types**: `<T extends Number & Comparable<T>>` — multiple bounds, class first.
- **Wildcards + PECS** (*Producer Extends, Consumer Super*):
  - `? extends T` → you can **read** T out (producer), can't add.
  - `? super T` → you can **write** T in (consumer), reads come back as `Object`.

## Enterprise example — PECS in a real signature
```java
// Reads (produces) from src, writes (consumes) into dst -> PECS
static <T> void copy(List<? super T> dst, List<? extends T> src) {
    for (T item : src) dst.add(item);
}

// Generic API wrapper with a bounded type:
public <T extends DomainEvent> void publish(T event, EventChannel<? super T> channel) {
    channel.send(event);
}
```
`Collections.max(Collection<? extends T>)` and `Collections.copy(List<? super T>, List<? extends T>)` are the canonical JDK examples.

## Invariance (the mental model)
`List<Dog>` is **not** a `List<Animal>` even though `Dog` is an `Animal` — generics are **invariant** to preserve type safety (an `Animal` list could accept a `Cat`). Wildcards reintroduce controlled covariance (`? extends`) or contravariance (`? super`). Arrays, by contrast, are **covariant** (`Dog[]` is an `Object[]`) and therefore only type-checked at runtime (`ArrayStoreException`) — a known wart generics avoid.

## Trade-offs
- Advantages: compile-time safety, self-documenting APIs, no manual casts.
- Disadvantages: erasure limits reflection/array creation; wildcard-heavy signatures hurt readability; interop with raw types produces unchecked warnings.

## Common mistakes (senior-level)
- Trying to create a generic array (`new T[]`) — use `Object[]` + cast or `Array.newInstance` with a `Class<T>`.
- Overusing wildcards where a single type parameter is clearer.
- Mixing raw types (`List` vs `List<T>`) and suppressing unchecked warnings blindly.
- Expecting runtime generic type info (it's erased — pass a `Class<T>` token if you need it, the "type token" pattern).

## Interview questions (staff+)
- What is type erasure and enumerate its consequences?
- Explain PECS with a concrete signature.
- Why is `List<Dog>` not a `List<Animal>`, and how do arrays differ (covariance)?
- What are bridge methods and why does the compiler generate them?
- How do you recover a type at runtime despite erasure (type token)?

## Related concepts
- [[Collections-Framework]]
- [[Streams]]
