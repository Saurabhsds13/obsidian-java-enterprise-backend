---
type: concept
domain: spring
topic: dependency-injection
difficulty: medium
status: inbox
tags: [spring]
---

# Dependency Injection

## Definition
Supplying a component's dependencies from outside rather than having it construct them. The concrete form of [[IoC]] in Spring.

## Why it matters
Constructor injection produces immutable, fully-initialized, easily-testable beans and makes required dependencies explicit.

## How it works
Three styles: constructor (preferred), setter, field. Spring resolves by type, disambiguated by `@Qualifier`/`@Primary`.

```java
@Service
public class OrderService {
    private final PaymentService payments;
    private final InventoryClient inventory;

    // Single constructor: @Autowired is optional
    public OrderService(PaymentService payments, InventoryClient inventory) {
        this.payments = payments;
        this.inventory = inventory;
    }
}
```

## Production usage
Constructor injection everywhere; it enables `final` fields, guards against `null`, and works without the Spring context in unit tests.

## Trade-offs
- Field injection is terse but hides dependencies and breaks plain-unit-testability.

## Common mistakes
- Field injection (`@Autowired` on fields).
- Circular dependencies — a design smell to refactor, not to patch with `@Lazy`.

## Interview questions
- Why is constructor injection preferred?
- How does Spring resolve two beans of the same type?

## Related concepts
- [[IoC]]
- [[Bean-Lifecycle]]
- [[ApplicationContext]]
