---
type: concept
domain: java
topic: functional
difficulty: easy
status: inbox
tags: [java]
---

# Optional

## Definition
A container that may or may not hold a non-null value, used to express "no result" explicitly instead of returning null.

## Why it matters
Makes absence part of the type signature, reducing `NullPointerException` and clarifying intent in APIs.

## How it works
```java
Optional<Account> found = repository.findById(id);
Account account = found.orElseThrow(() -> new AccountNotFoundException(id));

// Prefer functional style over isPresent/get:
String name = repository.findById(id)
    .map(Account::ownerName)
    .orElse("unknown");
```

## Production usage
Ideal as a **return type** for lookups that may find nothing. Spring Data repositories return `Optional<T>` from `findById`.

## Trade-offs
- Advantages: explicit absence, composable with `map`/`filter`/`flatMap`.
- Disadvantages: overhead if overused; not meant for fields or method parameters.

## Common mistakes
- Using `Optional` for fields or method arguments.
- Calling `get()` without checking presence (defeats the purpose) — use `orElse`/`orElseThrow`.
- Returning `null` for an `Optional`.

## Interview questions
- When should you use Optional and when not?
- Difference between `orElse` and `orElseGet`?

## Related concepts
- [[Streams]]
