---
type: interview
domain: java
topic: core-java
difficulty: easy
status: inbox
tags: [interview, java]
---

# Why is String immutable in Java?

## Question
> Why did Java make `String` immutable, and what benefits does it bring?

## Short answer
For safe sharing (including the String Pool), a cacheable hash code, thread safety, and security.

## Detailed answer
Immutability lets identical literals be interned and shared, caches `hashCode()` for fast use as [[HashMap]] keys, makes strings inherently thread-safe, and prevents a checked value (filename, URL, credential) from being altered after validation. The trade-off is that concatenation creates new objects, so use `StringBuilder` in loops.

## Example
```java
String a = "pay";
a.concat("ment");   // returns a new String; a is unchanged
```

## Production relevance
Avoid `+` in tight loops; prefer `StringBuilder`. Compare with `equals()`, not `==`.

## Common mistake
Believing `concat`/`replace` mutate the original.

## Follow-up questions
- How does the String Pool / `intern()` work?
- StringBuilder vs StringBuffer?

## Related concepts
- [[String-Immutability]]
- [[equals-and-hashCode]]
