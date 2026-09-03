---
type: concept
domain: spring
topic: spring-container
difficulty: medium
status: inbox
tags: [spring]
---

# ApplicationContext

## Definition
Spring's central IoC container. It instantiates, configures, and assembles beans, and adds enterprise features on top of the lower-level `BeanFactory`.

## Why it matters
It is the runtime that owns your beans. Knowing what it does explains startup behavior, bean scopes, events, and configuration resolution.

## How it works
- Reads configuration (component scanning, `@Configuration` classes).
- Builds the bean definition registry, resolves dependencies ([[Dependency-Injection]]), and manages the [[Bean-Lifecycle]].
- Adds: environment/property resolution, event publishing, internationalization, and `BeanPostProcessor` hooks.
- `BeanFactory` is lazy and minimal; `ApplicationContext` eagerly instantiates singletons and layers on the extras.

```java
var ctx = new AnnotationConfigApplicationContext(AppConfig.class);
PaymentService svc = ctx.getBean(PaymentService.class);
```

## Production usage
In Spring Boot, the context is created for you (`SpringApplication.run`). You rarely call `getBean` directly; you inject instead.

## Trade-offs
- Eager singleton creation surfaces wiring errors at startup (good) but adds startup time.

## Common mistakes
- Manually pulling beans with `getBean` instead of injecting.

## Interview questions
- BeanFactory vs ApplicationContext?
- What happens during context startup?

## Related concepts
- [[IoC]]
- [[Bean-Lifecycle]]
- [[Spring-Boot-Auto-Configuration]]
