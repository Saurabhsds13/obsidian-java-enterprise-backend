---
type: concept
domain: spring
topic: spring-mvc
difficulty: easy
status: inbox
tags: [spring]
---

# REST Controller

## Definition
A `@RestController` maps HTTP requests to handler methods and serializes return values (usually JSON) directly to the response body.

## Why it matters
It's the entry point for REST APIs. Clean controllers keep the [[REST]] contract stable and testable.

## How it works
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService service;
    public OrderController(OrderService service) { this.service = service; }

    @GetMapping("/{id}")
    public ResponseEntity<OrderDto> get(@PathVariable long id) {
        return ResponseEntity.ok(service.get(id));
    }

    @PostMapping
    public ResponseEntity<OrderDto> create(@Valid @RequestBody CreateOrderRequest req) {
        OrderDto created = service.create(req);
        return ResponseEntity.created(URI.create("/api/v1/orders/" + created.id())).body(created);
    }
}
```

## Production usage
Use DTOs (not entities), `@Valid` for input ([[Validation]]), `ResponseEntity` for status/headers, and consistent error bodies via [[Exception-Handling]]. Version the path ([[API-Versioning]]).

## Trade-offs
- Returning `ResponseEntity` everywhere is explicit but verbose; returning the body directly is terser.

## Common mistakes
- Exposing JPA entities directly.
- Inconsistent status codes.

## Interview questions
- `@Controller` vs `@RestController`?
- How do you return 201 with a Location header?

## Related concepts
- [[Spring-MVC]]
- [[REST]]
- [[Validation]]
- [[Exception-Handling]]
