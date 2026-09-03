---
type: concept
domain: java
topic: functional
difficulty: medium
status: inbox
tags: [java]
---

# Optional

## Definition
A container (Java 8+) that either holds a non-null value or is empty, used to make "no result" an explicit, type-level outcome instead of a bare `null`. It is a value-based class — treat it as immutable and don't rely on its identity.

## Why it matters
It moves absence into the method signature, reducing `NullPointerException` and clarifying API contracts. Interviewers use it to check that you know its *intended* scope (return types) versus the misuses (fields, parameters, collections).

## How it works — API semantics
```java
Optional<Account> found = repo.findById(id);           // absence is explicit

// Functional composition (preferred over isPresent/get):
String owner = repo.findById(id)
    .map(Account::owner)               // transform if present
    .filter(o -> o.active())           // drop if predicate fails
    .map(Owner::name)
    .orElse("unknown");

Account a = repo.findById(id)
    .orElseThrow(() -> new NotFoundException(id));   // fail with a meaningful exception
```
Key distinction: **`orElse(x)`** always evaluates `x` (even when present) — bad if `x` is expensive; **`orElseGet(supplier)`** is lazy. `flatMap` avoids `Optional<Optional<T>>` when the mapper itself returns an `Optional`.

## Where it belongs (and doesn't)
| Use | Verdict |
|-----|---------|
| Method **return type** for "maybe absent" | ✅ intended use |
| Field on an entity/DTO | ❌ not `Serializable`, adds overhead, JPA can't map it |
| Method **parameter** | ❌ use overloads or `@Nullable`; callers must wrap needlessly |
| Collection element (`List<Optional<T>>`) | ❌ use an empty collection or filter out absents |
| Wrapping a primitive | prefer `OptionalInt`/`OptionalLong`/`OptionalDouble` (avoid boxing) |

## Enterprise example
```java
public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByEmail(String email);   // Spring Data returns Optional
}
// Never return null from an Optional-typed method; return Optional.empty().
```

## Trade-offs
- Advantages: explicit absence, composable, self-documenting.
- Disadvantages: allocation/overhead if overused in hot paths; not for serialization; verbosity if you fall back to `isPresent()/get()`.

## Common mistakes (senior-level)
- `optional.get()` without checking → defeats the purpose (`orElseThrow` with a real exception instead).
- `orElse(expensiveCall())` where `orElseGet` should be used.
- `Optional` as a field/parameter or returning `null` for an `Optional`.
- `optional.isPresent() ? optional.get() : default` instead of `map`/`orElse`.

## Interview questions (staff+)
- Where should `Optional` be used and where must it not?
- `orElse` vs `orElseGet` vs `orElseThrow` — the evaluation difference.
- Why not use it for fields (JPA/serialization)?
- `map` vs `flatMap` on Optional.

## Related concepts
- [[Streams]]
