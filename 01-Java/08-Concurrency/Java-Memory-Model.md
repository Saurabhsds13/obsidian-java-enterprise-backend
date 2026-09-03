---
type: concept
domain: java
topic: concurrency
difficulty: hard
status: inbox
tags: [java]
---

# Java Memory Model

## Definition
The JMM specifies how and when writes by one thread become visible to others, and which reorderings the compiler/CPU may perform. Its core rule is the **happens-before** relationship.

## Why it matters
Without it, concurrent code "works on my machine" but fails unpredictably. It is the foundation beneath [[volatile]], [[synchronized]], and concurrent collections.

## How it works
A write is guaranteed visible to a read only if there is a happens-before edge between them. Key edges:
- Program order within a single thread.
- Unlock of a monitor happens-before subsequent lock ([[synchronized]]).
- A `volatile` write happens-before subsequent `volatile` reads of that field.
- `Thread.start()` happens-before the started thread's actions; a thread's actions happen-before another thread's `join()`.

```java
volatile boolean ready = false;
int data;

// Thread A
data = 42;      // (1)
ready = true;   // (2) volatile write

// Thread B
if (ready) {    // (3) volatile read
    use(data);  // guaranteed to see 42 due to happens-before (2)->(3)
}
```

## Production usage
Explains why unsynchronized data races are undefined behavior, and why "just add volatile" is sometimes right (flags) and sometimes wrong (counters).

## Trade-offs
- Reasoning about happens-before is hard; prefer high-level constructs (concurrent collections, executors) that encapsulate it.

## Common mistakes
- Assuming plain field writes are visible across threads.
- Treating `volatile` as a lock.

## Interview questions
- Explain happens-before.
- Why can a data race produce impossible-looking values?

## Related concepts
- [[volatile]]
- [[synchronized]]
- [[ConcurrentHashMap]]
