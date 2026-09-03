---
type: concept
domain: java
topic: core-java
difficulty: hard
status: inbox
tags: [java]
---

# equals() and hashCode()

## Definition
`equals()` defines logical equality; `hashCode()` returns an int used by hash-based collections to bucket objects. `Object`'s defaults use identity (reference equality and an identity hash). Overriding one without the other, or using mutable fields, breaks the contract that hash collections depend on.

## Why it matters
[[HashMap]], `HashSet`, and [[ConcurrentHashMap]] correctness rests entirely on this contract. Violations cause "lost" entries, phantom duplicates, and heisenbugs — and it's a favorite interview probe because JPA entity identity ([[Hibernate]]) makes it genuinely subtle.

## How it works — the contract
1. **Consistency with equals**: `a.equals(b)` ⇒ `a.hashCode() == b.hashCode()`.
2. **Collision allowed**: equal hash codes do *not* imply equality.
3. **Reflexive, symmetric, transitive, consistent**; `x.equals(null)` is false.
4. **Stability**: hash code must not change while the object is a key (i.e. use immutable fields).

Hash collections use the hash to find the *bucket*, then `equals()` to find the *entry* within it. Break rule 1 and `get` looks in the wrong bucket → miss.

## Correct implementation
```java
public final class Money {
    private final long cents;
    private final String currency;
    // constructor omitted

    @Override public boolean equals(Object o) {
        if (this == o) return true;                 // fast path
        if (!(o instanceof Money m)) return false;  // pattern match; handles null + type
        return cents == m.cents && currency.equals(m.currency);
    }
    @Override public int hashCode() {
        return Objects.hash(cents, currency);       // consistent with equals
    }
}
```
Prefer a Java `record` for value types — it generates a correct, immutable `equals`/`hashCode`/`toString` for you.

## The symmetry trap with inheritance
An `equals` that uses `instanceof` across a subclass can break **symmetry** (`a.equals(b)` ≠ `b.equals(a)`). Rule of thumb: don't add value fields to a subclass of a class that supports value equality; favor composition, or use `getClass()` comparison (which then breaks Liskov for proxies). This is why value objects are usually `final`.

## The JPA/Hibernate landmine
- A generated `@Id` is **null before persist**. If `hashCode` uses it, an entity's hash changes after save → it "disappears" from a `HashSet` it was added to while transient.
- Guidance: base `equals`/`hashCode` on a **stable business key** (natural key) if one exists; otherwise use a UUID assigned at construction. Never use a Hibernate-managed collection's contents.

## Trade-offs
- Using all fields = precise equality but fragile if any field is mutable.
- Using a business key = stable and DB-friendly but requires one to exist.

## Common mistakes (senior-level)
- Overriding `equals` but not `hashCode` (or vice versa) → broken map/set behavior.
- Mutable fields in the hash → key becomes unreachable after mutation.
- Generated-`@Id` in `hashCode` for JPA entities.
- Asymmetric `equals` via subclassing.
- Expensive `equals`/`hashCode` on hot keys (hash each lookup) — keep them cheap; cache if needed (`String` caches its hash).

## Interview questions (staff+)
- State the full contract and what breaks if you violate rule 1.
- Why is a HashSet the classic place a broken contract surfaces?
- How should JPA entities implement equals/hashCode, and why not the generated id?
- Why do value objects tend to be `final` (symmetry)?
- record vs hand-written equals/hashCode?

## Related concepts
- [[HashMap]]
- [[ConcurrentHashMap]]
- [[String-Immutability]]
- [[Hibernate]]
