---
type: concept
domain: engineering-practices
topic: design
difficulty: medium
status: inbox
tags: [engineering-practices]
---

# Design Patterns

## Definition
Reusable solutions to recurring design problems, grouped as creational, structural, and behavioral.

## Why it matters
Patterns give a shared vocabulary and proven structures, heavily used in LLD interviews and Spring itself.

## Common patterns (with Java/Spring context)
- **Creational**: Factory, Builder, Singleton (Spring beans are effectively singletons).
- **Structural**: Adapter, Decorator, Proxy (Spring AOP uses proxies — see [[Spring-AOP]]), Facade.
- **Behavioral**: Strategy (pluggable algorithms — see [[Parking-Lot]]), Observer (events), Template Method, State.

## Example
```java
// Strategy: swap behavior without changing the caller
interface ShippingCost { BigDecimal cost(Order o); }
class Checkout {
    private final ShippingCost shipping;
    Checkout(ShippingCost shipping) { this.shipping = shipping; }
}
```

## Production usage
Spring is full of patterns: proxy (AOP/transactions), template (`JdbcTemplate`), factory (bean factories), observer (application events). Use patterns to clarify intent, not to show off.

## Trade-offs
- Patterns add structure and flexibility but can over-engineer simple problems.

## Common mistakes
- Forcing a pattern where a plain method/class would do.

## Interview questions
- Where does Spring use the proxy/template/strategy pattern?
- When would you use Strategy vs simple polymorphism?

## Related concepts
- [[SOLID]]
- [[Spring-AOP]]
- [[Parking-Lot]]
