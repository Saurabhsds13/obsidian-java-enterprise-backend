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
`String` instances cannot change after construction. The backing storage is `private final` (a `char[]` pre-Java 9, a `byte[]` + coder since Java 9's *compact strings*), and no method mutates it — operations like `concat`/`replace`/`substring` return new `String`s.

## Why it matters
Immutability is what makes the **String Pool**, safe sharing across threads, a cached hash code, and secure APIs possible. It's a classic "why" interview question that opens into the JMM, hashing, and memory.

## How it works — the mechanism
- **Final backing array + no mutators** → the value is fixed for the object's life.
- **String Pool (interning)**: string *literals* are interned in a shared pool (stored in the heap since Java 7), so equal literals reference the same object. `new String("x")` forces a distinct heap object; `.intern()` returns the pooled canonical instance.
- **Cached hash**: `String` caches its `hashCode()` after first computation (safe precisely because it's immutable) — this makes strings fast [[HashMap]] keys.
- **Compact strings (Java 9+)**: Latin-1 strings store 1 byte/char instead of 2, roughly halving memory for typical ASCII-heavy workloads.

```java
String a = "pay";
String b = "pay";              // same pooled instance
String c = new String("pay");  // distinct object
a == b;            // true  (pooled)
a == c;            // false (different object)
a.equals(c);       // true  (always compare content with equals)
```

## Why immutable specifically (the four reasons)
1. **Pooling** — safe to share one instance for identical literals only because no one can mutate it.
2. **Thread safety** — freely shared without synchronization (see [[Java-Memory-Model]]).
3. **Cached hash** — enables fast, stable [[HashMap]]/[[equals-and-hashCode]] keys.
4. **Security** — a validated filename/URL/host can't be changed after the security check (TOCTOU protection).

## Enterprise example — build, don't concatenate
```java
// BAD: N intermediate String objects in a loop (O(n^2) copying)
String csv = "";
for (var row : rows) csv += row + ",";

// GOOD: single mutable buffer
StringBuilder sb = new StringBuilder(rows.size() * 16);
for (var row : rows) sb.append(row).append(',');
String csv = sb.toString();
```
`StringBuilder` (not thread-safe, fast) vs `StringBuffer` (synchronized, legacy). The compiler already turns simple `+` into `StringBuilder`, but not across loop iterations — hence the manual builder in loops.

## Trade-offs
- Advantages: safety, pooling, cacheable hash, cheap sharing.
- Disadvantages: transformations allocate new objects → use `StringBuilder` for heavy concatenation; over-`intern()` pressures the pool and can retain memory.

## Common mistakes (senior-level)
- `==` instead of `equals()` for content comparison.
- `+=` concatenation in loops (garbage + O(n²)).
- Overusing `intern()` (pool pressure, GC retention).
- Holding a `substring` of a huge string expecting it to share memory (modern JDKs copy, so it doesn't leak the parent — but don't rely on old sharing behavior).

## Interview questions (staff+)
- Why is `String` immutable — give the four concrete benefits.
- How does the String Pool work; literal vs `new String` vs `intern()`?
- What are compact strings and why do they matter?
- StringBuilder vs StringBuffer, and what does `s1 + s2` compile to?
- How does immutability enable the cached hash code?

## Related concepts
- [[equals-and-hashCode]]
- [[HashMap]]
- [[Java-Memory-Model]]
- [[Why-is-String-immutable]]
