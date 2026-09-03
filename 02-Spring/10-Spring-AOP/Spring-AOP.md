---
type: concept
domain: spring
topic: aop
difficulty: medium
status: inbox
tags: [spring]
---

# Spring AOP

## Definition
Aspect-Oriented Programming modularizes cross-cutting concerns (transactions, security, logging, metrics) into **aspects** applied around method executions via proxies.

## Why it matters
It underpins [[Transactional]], method security, and `@Cacheable`. Understanding proxying explains why some annotations "don't work" on self-calls.

## How it works
- Terms: **aspect** (the module), **advice** (the action: before/after/around), **pointcut** (where it applies), **join point** (a method execution).
- Spring uses **proxies**: JDK dynamic proxies (interface-based) or CGLIB (subclass-based).
- The proxy intercepts external calls; a call from within the same object (`this.method()`) bypasses the proxy.

```java
@Aspect @Component
public class TimingAspect {
    @Around("@annotation(Timed)")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try { return pjp.proceed(); }
        finally { metrics.record(System.nanoTime() - start); }
    }
}
```

## Production usage
Prefer built-in aspects (`@Transactional`, `@Cacheable`, method security) over custom ones. Custom aspects suit auditing, timing, and retries.

## Trade-offs
- Powerful but implicit; overuse makes control flow hard to follow.

## Common mistakes
- Expecting advice on self-invoked or private methods.

## Interview questions
- How does Spring AOP work (proxies)?
- Why does self-invocation bypass `@Transactional`?

## Related concepts
- [[Transactional]]
- [[Bean-Lifecycle]]
- [[Spring-Security]]
