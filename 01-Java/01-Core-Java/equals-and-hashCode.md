---
type: concept
domain: java
topic: core-java
difficulty: medium
status: inbox
tags: [java]
---

# equals() and hashCode()

## Definition
`equals()` defines logical equality between two objects; `hashCode()` returns an int used by hash-based collections to bucket objects. They form a contract that must be honored together.

## Why it matters
[[HashMap]], `HashSet`, and [[ConcurrentHashMap]] rely on this contract. Breaking it causes lost entries, duplicates, and subtle bugs that are hard to trace.

## How it works
The contract:
1. If `a.equals(b)` is true, then `a.hashCode() == b.hashCode()`.
2. Equal hash codes do **not** require equality (collisions are allowed).
3. Both must be consistent across calls unless the object's state changes.

```java
public final class Money {
    private final long cents;
    private final String currency;

    // constructor omitted

    @Override public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money m)) return false;
        return cents == m.cents && currency.equals(m.currency);
    }

    @Override public int hashCode() {
        return Objects.hash(cents, currency);
    }
}
```

## Production usage
Use immutable fields in `equals`/`hashCode`. For JPA entities this is subtle — prefer a stable business key or a natural ID rather than a database-generated ID that is null before persist (see [[Hibernate]]).

## Trade-offs
- Including mutable fields makes objects "lose themselves" in a `HashSet` after mutation.

## Common mistakes
- Overriding `equals` but not `hashCode` (or vice versa).
- Using generated IDs that are null pre-persist in entity `hashCode`.

## Interview questions
- What is the equals/hashCode contract?
- What happens if you override equals but not hashCode?

## Related concepts
- [[HashMap]]
- [[String-Immutability]]
- [[ConcurrentHashMap]]
