---
type: concept
domain: spring
topic: spring-mvc
difficulty: medium
status: inbox
tags: [spring]
---

# Exception Handling

## Definition
Centralized translation of exceptions into consistent HTTP responses using `@ExceptionHandler` and `@ControllerAdvice`.

## Why it matters
Consistent, informative error responses are part of a good [[REST]] contract and avoid leaking stack traces.

## How it works
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ProblemDetail notFound(EntityNotFoundException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        pd.setTitle("Resource not found");
        return pd;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail invalid(MethodArgumentNotValidException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation failed");
    }
}
```
Spring 6 supports RFC 7807 `ProblemDetail` for standardized error bodies.

## Production usage
One advice class per app (or module) mapping domain exceptions to status codes. Never expose internal messages/stack traces to clients; log them server-side with a correlation ID ([[Observability]]).

## Trade-offs
- Over-catching hides real bugs; map deliberately.

## Common mistakes
- Returning 500 for client errors (should be 4xx).
- Leaking exception details to clients.

## Interview questions
- How does `@ControllerAdvice` work?
- What is RFC 7807 / ProblemDetail?

## Related concepts
- [[Spring-MVC]]
- [[Validation]]
- [[REST]]
