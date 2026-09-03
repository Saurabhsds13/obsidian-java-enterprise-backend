---
type: concept
domain: java
topic: strings
difficulty: medium
status: inbox
tags: [java]
---

# String Immutability

## Definition
Java `String` objects cannot be modified after creation. Any operation that appears to change a string (e.g. `concat`, `replace`) returns a new `String`.

## Why it matters
Immutability enables the **String Pool** (interning), safe sharing across threads, and safe use of strings as [[HashMap]] keys and in security-sensitive APIs.

## How it works
`String` wraps a `private final` character array and exposes no mutators. The JVM interns string literals in a shared pool, so identical literals reference the same object.

```java
String a = "pay";
String b = "pay";        // same pooled instance
System.out.println(a == b); // true

String c = new String("pay"); // new heap object
System.out.println(a == c);   // false
System.out.println(a.equals(c)); // true — always compare with equals()
```

## Why immutable specifically
- **Caching hash code**: `hashCode()` is computed once and cached, making strings fast [[HashMap]] keys (see [[equals-and-hashCode]]).
- **Thread safety**: shared freely without synchronization.
- **Security**: a filename/URL passed to a method can't be mutated after a security check.

## Production usage
Build strings in loops with `StringBuilder` (not `+`) to avoid creating many intermediate objects. Use `String.intern()` sparingly; it can pressure the pool.

## Trade-offs
- Advantages: safety, cacheable hash, pooling.
- Disadvantages: repeated concatenation creates garbage — use `StringBuilder`.

## Common mistakes
- Comparing strings with `==` instead of `equals()`.
- Concatenating in tight loops with `+`.

## Interview questions
- Why is String immutable in Java?
- How does the String Pool work? What does `intern()` do?

## Related concepts
- [[equals-and-hashCode]]
- [[HashMap]]
- [[Why-is-String-immutable]]
