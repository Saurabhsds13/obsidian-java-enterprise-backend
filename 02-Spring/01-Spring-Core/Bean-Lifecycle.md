---
type: concept
domain: spring
topic: spring-core
difficulty: medium
status: inbox
tags: [spring]
---

# Bean Lifecycle

## Definition
The sequence of phases a Spring bean goes through: instantiation, dependency injection, initialization callbacks, use, and destruction.

## Why it matters
Lifecycle hooks are where you open/close resources (pools, connections) and validate configuration at the right time.

## How it works
```text
Instantiate -> Populate dependencies -> Aware callbacks
   -> BeanPostProcessor.before -> @PostConstruct / afterPropertiesSet
   -> BeanPostProcessor.after (proxies created here, e.g. @Transactional/AOP)
   -> [bean in use] -> @PreDestroy / destroy()
```

```java
@Component
public class CacheWarmer {
    @PostConstruct void init()  { /* warm cache after DI */ }
    @PreDestroy   void close()  { /* release resources on shutdown */ }
}
```

## Production usage
`@PostConstruct` for setup that needs injected dependencies; `@PreDestroy` for graceful shutdown. AOP proxies ([[Spring-AOP]], [[Transactional]]) are applied by a `BeanPostProcessor` — which is why self-invocation bypasses them.

## Trade-offs
- Heavy `@PostConstruct` work slows startup and can fail readiness.

## Common mistakes
- Assuming dependencies are available in the constructor for post-wiring work (use `@PostConstruct`).

## Interview questions
- Order of lifecycle callbacks?
- Where are AOP/transaction proxies created?

## Related concepts
- [[ApplicationContext]]
- [[Dependency-Injection]]
- [[Spring-AOP]]
- [[Transactional]]
