---
type: concept
domain: java
topic: core-java
difficulty: easy
status: inbox
tags: [java]
---

# JDK vs JRE

## Definition
- **JDK** (Java Development Kit): tools to *develop* Java — compiler (`javac`), debugger, `jar`, and the JRE.
- **JRE** (Java Runtime Environment): everything needed to *run* Java — the [[JVM]] plus core libraries. Historically shipped separately; modern distributions bundle a runtime image.

## Why it matters
Knowing the difference clarifies what you ship to production (a runtime) versus what you build with (a full kit), and it explains base image choices in Docker.

## How it works
```text
JDK = JRE + development tools (javac, jdb, jar, javadoc...)
JRE = JVM + standard class libraries
JVM = the execution engine
```
Since Java 11, standalone public JREs were discontinued; you produce a slim runtime with `jlink`, or use a JDK base image.

## Example
```dockerfile
# Build stage uses full JDK
FROM eclipse-temurin:21-jdk AS build
# Runtime stage uses a smaller runtime image
FROM eclipse-temurin:21-jre
```

## Production usage
CI/build agents need the JDK; runtime containers can use a JRE or a `jlink`-generated custom runtime to shrink image size and attack surface.

## Trade-offs
- JRE-only images are smaller and expose fewer tools, but you can't compile on them.

## Common mistakes
- Shipping the full JDK to production when only a runtime is needed.

## Interview questions
- What is the difference between JDK, JRE, and JVM?
- How do you produce a minimal Java runtime today?

## Related concepts
- [[JVM]]
- [[Garbage-Collection]]
