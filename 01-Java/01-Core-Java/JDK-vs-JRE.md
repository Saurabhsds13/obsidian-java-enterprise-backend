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
- **JDK** (Java Development Kit): everything needed to *develop* — compiler (`javac`), tools (`jar`, `javadoc`, `jlink`, `jcmd`, `jstack`), plus a runtime.
- **JRE** (Java Runtime Environment): everything needed to *run* — the [[JVM]] plus the standard class library. Standalone public JREs were discontinued after Java 8/11; you now ship a JDK or a `jlink`-built custom runtime.
- **JVM**: the execution engine inside both.

## Why it matters
It clarifies what you build with vs what you ship, drives Docker base-image and image-size decisions, and is a quick "do you understand the platform layering" screen.

## How it works
```
JDK = JRE + dev tools (javac, jdb, jar, javadoc, jlink, diagnostics)
JRE = JVM + standard libraries
JVM = bytecode execution engine (see [[JVM]])
```
Since Java 9's module system, `jlink` assembles a **minimal custom runtime** containing only the modules your app needs — smaller image, smaller attack surface. GraalVM `native-image` goes further, producing a standalone native binary (no JVM shipped) with fast startup, trading away JIT peak throughput and some dynamic features.

## Enterprise example — multi-stage Docker
```dockerfile
# Build with the full JDK
FROM eclipse-temurin:21-jdk AS build
COPY . . 
RUN ./mvnw -q package

# Ship a slim runtime image (JRE-class)
FROM eclipse-temurin:21-jre
COPY --from=build /app/target/app.jar /app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

## Trade-offs
- Full JDK image: can compile/diagnose in-container, larger, bigger attack surface.
- JRE / `jlink` runtime: smaller and safer, but can't compile and has fewer tools (harder live debugging).
- `native-image`: fastest startup + lowest memory, but loses JIT peak throughput, needs closed-world config for reflection.

## Common mistakes (senior-level)
- Shipping the full JDK to production when a runtime suffices (larger image, more CVEs).
- Removing all diagnostic tools then being unable to `jstack`/`jcmd` a stuck prod pod — keep a debug sidecar or a JDK image for triage.
- Forgetting `native-image` needs explicit reflection/resource config.

## Interview questions (staff+)
- JDK vs JRE vs JVM — what's in each?
- How do you produce a minimal Java runtime today (`jlink`), and why?
- Trade-offs of GraalVM native-image vs a JVM runtime.
- Why might you still ship a JDK image to prod?

## Related concepts
- [[JVM]]
- [[Garbage-Collection]]
