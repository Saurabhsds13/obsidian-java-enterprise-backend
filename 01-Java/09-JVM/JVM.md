---
type: concept
domain: java
topic: jvm
difficulty: medium
status: inbox
tags: [java]
---

# JVM

## Definition
The Java Virtual Machine is an abstract computing machine that executes Java bytecode. It provides platform independence ("write once, run anywhere"), automatic memory management, and runtime services such as JIT compilation and garbage collection.

## Why it matters
The JVM is why Java code runs unchanged across operating systems, and it is the layer where performance, memory, and GC behavior are decided. Understanding it separates engineers who *use* Java from those who can *tune and debug* it in production.

## How it works
1. `.java` source is compiled by `javac` into `.class` files containing **bytecode**.
2. The **class loader** loads classes into memory (see [[JDK-vs-JRE]]).
3. Bytecode runs on the **execution engine**: initially interpreted, then hot paths are compiled to native code by the **JIT** compiler.
4. Runtime data areas hold program state:
   - **Heap** — objects (shared, GC-managed). See [[Garbage-Collection]].
   - **Stack** — one per thread, holds frames with local variables.
   - **Metaspace** — class metadata (off-heap since Java 8).
   - **PC register** and **native method stack**.

```text
Source (.java) --javac--> Bytecode (.class) --ClassLoader--> JVM
      Execution Engine (Interpreter + JIT) -> native CPU instructions
```

## Example
```bash
javac PaymentService.java   # produces PaymentService.class (bytecode)
java PaymentService         # JVM loads, verifies, and executes bytecode
```

## Production usage
JVM flags tune heap and GC for services: `-Xms`/`-Xmx` (heap size), `-XX:+UseG1GC`, `-XX:MaxMetaspaceSize`. Heap dumps (`jmap`) and thread dumps (`jstack`) are core production debugging tools.

## Trade-offs
- Advantages: portability, mature GC, strong tooling and observability.
- Disadvantages: startup/warmup cost (JIT), memory overhead vs native binaries.
- When NOT to ignore it: latency-sensitive services need GC and warmup tuning.

## Common mistakes
- Assuming `-Xmx` is the total process memory (Metaspace, thread stacks, and native buffers live outside the heap).
- Confusing interpreter-only behavior in benchmarks with warmed-up JIT performance.

## Interview questions
- What are the runtime data areas of the JVM?
- Heap vs stack vs Metaspace — what lives where?
- What does the JIT do and when does it kick in?

## Related concepts
- [[JDK-vs-JRE]]
- [[Garbage-Collection]]
- [[Java-Memory-Model]]
