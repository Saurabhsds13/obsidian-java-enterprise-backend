---
type: concept
domain: spring
topic: spring-core
difficulty: medium
status: inbox
tags: [spring]
---

# IoC (Inversion of Control)

## Definition
A principle where object creation and wiring are handed to a container instead of being done by the objects themselves. Spring's container owns the lifecycle of beans.

## Why it matters
IoC decouples components from their dependencies' construction, enabling testability, configuration flexibility, and consistent lifecycle management.

## How it works
You declare *what* you need; the container decides *how* to provide it. [[Dependency-Injection]] is the mechanism. The [[ApplicationContext]] is the container that reads configuration (annotations, Java config) and builds the bean graph.

```java
@Service
public class PaymentService {
    private final PaymentGateway gateway;   // dependency, not created here
    public PaymentService(PaymentGateway gateway) { this.gateway = gateway; }
}
```

## Production usage
Nearly all Spring apps rely on IoC to wire controllers → services → repositories, and to swap implementations per [[Spring-Boot-Auto-Configuration|profile/config]].

## Trade-offs
- Advantages: loose coupling, testability, centralized config.
- Disadvantages: indirection can obscure wiring; startup cost.

## Common mistakes
- Fighting the container with `new` inside beans.
- Field injection instead of constructor injection.

## Interview questions
- What is IoC and how does Spring implement it?
- IoC vs Dependency Injection — are they the same?

## Related concepts
- [[Dependency-Injection]]
- [[ApplicationContext]]
- [[Bean-Lifecycle]]
