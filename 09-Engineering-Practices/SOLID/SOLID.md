---
type: concept
domain: engineering-practices
topic: design
difficulty: medium
status: inbox
tags: [engineering-practices]
---

# SOLID

## Definition
Five object-oriented design principles for maintainable, extensible code.

## Why it matters
SOLID guides class design so systems tolerate change — central to LLD interviews ([[Parking-Lot]]) and real codebases.

## The principles
- **S — Single Responsibility**: a class has one reason to change.
- **O — Open/Closed**: open for extension, closed for modification (add behavior via new types, not edits).
- **L — Liskov Substitution**: subtypes must be usable wherever the base type is expected.
- **I — Interface Segregation**: prefer small, focused interfaces over fat ones.
- **D — Dependency Inversion**: depend on abstractions, not concretions.

## Example
```java
// OCP + DIP: add pricing strategies without changing the consumer
interface PricingStrategy { BigDecimal price(Order o); }
class Checkout {
    private final PricingStrategy pricing;   // depends on abstraction
    Checkout(PricingStrategy pricing) { this.pricing = pricing; }
}
```

## Production usage
Drives DI ([[Dependency-Injection]]), strategy-based extensibility, and testable seams. Apply pragmatically — don't over-abstract.

## Trade-offs
- Improves flexibility/testability, but over-application adds needless indirection (violates KISS/YAGNI).

## Common mistakes
- God classes (SRP); rigid `if/else` type checks instead of polymorphism (OCP).

## Interview questions
- Give a real example of each principle.
- How does DIP relate to Spring DI?

## Related concepts
- [[Design-Patterns]]
- [[Dependency-Injection]]
- [[Parking-Lot]]
