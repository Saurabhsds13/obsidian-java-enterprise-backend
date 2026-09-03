---
type: concept
domain: spring
topic: aop
difficulty: hard
status: inbox
tags: [spring]
---

# Spring AOP

## Definition
Spring AOP modularizes cross-cutting concerns (transactions, security, caching, metrics, retries) by weaving **advice** around method executions using **runtime proxies**. Unlike full AspectJ (compile/load-time bytecode weaving of any join point), Spring AOP is **proxy-based** and therefore only intercepts **public method calls that pass through the proxy** — a limitation that explains most "why didn't my annotation work" bugs.

## Why it matters
`@Transactional`, `@Cacheable`, `@Async`, `@PreAuthorize`, and Resilience4j annotations are *all* implemented as AOP advice. Understanding the proxy mechanism is the difference between using these annotations and *debugging* them under pressure — the classic self-invocation failure is a top-5 Spring interview question.

## How it works — the mechanism
- Terms: **join point** (a method execution), **pointcut** (predicate selecting join points), **advice** (before/after/around action), **aspect** (pointcut + advice), **weaving** (applying it — at runtime for Spring).
- A `BeanPostProcessor` (`AbstractAutoProxyCreator`) inspects each bean during the [[Bean-Lifecycle]] and, if any advisor's pointcut matches, replaces the bean in the context with a **proxy** that wraps it.
- **Two proxy strategies:**
  | | JDK dynamic proxy | CGLIB |
  |--|-------------------|-------|
  | Basis | implements the bean's **interfaces** | generates a **subclass** |
  | Requires | at least one interface | no interface (default in Boot since 2.0) |
  | Can't proxy | `final` classes/methods → silently not advised | `final` classes/methods, `private` |
- The proxy applies advice, then delegates to the **target** instance.

## The self-invocation problem (the #1 gotcha)
```java
@Service
public class ReportService {
    public void run() {
        generate();          // 'this.generate()' -> calls the TARGET directly,
    }                        // BYPASSING the proxy -> @Cacheable/@Transactional DO NOT apply
    @Cacheable("reports")
    public Report generate() { ... }
}
```
The proxy only wraps *external* calls. `this.generate()` goes straight to the target. Fixes:
1. Move `generate()` to a **separate bean** (cleanest — respects the boundary).
2. **Self-inject** the proxy (`@Autowired ReportService self;` then `self.generate()`), or use `AopContext.currentProxy()` (needs `exposeProxy = true`).
3. Switch to **AspectJ load-time weaving** (advises the real object, no proxy) — heavier, rarely worth it.

Same root cause blocks advice on `private`/`final`/non-public methods and calls from a constructor/`@PostConstruct` (proxy not fully ready).

## Enterprise example — a custom timing/metrics aspect
```java
@Aspect @Component
public class TimedAspect {
    private final MeterRegistry meters;
    public TimedAspect(MeterRegistry meters) { this.meters = meters; }

    @Around("@annotation(timed)")
    public Object time(ProceedingJoinPoint pjp, Timed timed) throws Throwable {
        Timer.Sample s = Timer.start(meters);
        try {
            return pjp.proceed();                    // invoke the advised method
        } finally {
            s.stop(meters.timer(timed.value()));     // always record, even on exception
        }
    }
}
```

## Advice ordering
Multiple aspects on one method order by `@Order`/`Ordered` (lower value = outer). This matters when combining, e.g., `@Transactional` and a retry aspect — retry **outside** the transaction (retry re-opens a fresh tx) vs inside (retries within a doomed, marked-rollback tx) is a real design decision.

## Trade-offs
- Advantages: keeps business code clean; declarative; powers Spring's most-used features.
- Disadvantages: proxy boundary is invisible in the source (surprising control flow); self-invocation/`final`/`private` limitations; a stack of aspects complicates debugging.

## Common mistakes (senior-level)
- Expecting advice on self-invoked, `private`, or `final` methods.
- Getting aspect order wrong (retry vs transaction; security vs caching).
- Assuming Spring AOP can advise arbitrary join points like AspectJ (it can't — method execution on Spring beans only).
- Invoking advised methods from a constructor/`@PostConstruct`.

## Interview questions (staff+)
- How does Spring AOP work, and how does it differ from AspectJ?
- JDK dynamic proxy vs CGLIB — when is each used and what can't each proxy?
- Explain self-invocation and three ways to fix it.
- If you have `@Transactional` + `@Retryable` on a method, what order do you want and why?
- Where in the bean lifecycle is the proxy created?

## Related concepts
- [[Transactional]]
- [[Bean-Lifecycle]]
- [[Spring-Security]]
