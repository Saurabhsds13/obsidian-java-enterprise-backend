---
type: concept
domain: spring
topic: spring-validation
difficulty: easy
status: inbox
tags: [spring]
---

# Validation

## Definition
Declarative input validation using Jakarta Bean Validation annotations (`@NotNull`, `@Size`, `@Email`) triggered by `@Valid`.

## Why it matters
Rejecting bad input at the boundary prevents corrupt state and is a first line of [[API-Security|API security]].

## How it works
```java
public record CreateOrderRequest(
    @NotBlank String customerId,
    @Positive BigDecimal amount,
    @Size(max = 500) String note) {}

@PostMapping
public ResponseEntity<Void> create(@Valid @RequestBody CreateOrderRequest req) { ... }
```
A failed `@Valid` throws `MethodArgumentNotValidException`, handled centrally ([[Exception-Handling]]). Use `@Validated` for method-level and group validation.

## Production usage
Validate DTOs at the controller boundary; keep domain invariants in the domain model too. Return field-level error details in the response.

## Trade-offs
- Annotations cover structural rules; complex cross-field rules need custom validators.

## Common mistakes
- Forgetting `@Valid` (validation silently skipped).
- Validating entities instead of request DTOs.

## Interview questions
- How does `@Valid` trigger validation?
- `@Valid` vs `@Validated`?

## Related concepts
- [[Exception-Handling]]
- [[REST-Controller]]
