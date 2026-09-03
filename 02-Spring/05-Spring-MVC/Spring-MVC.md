---
type: concept
domain: spring
topic: spring-mvc
difficulty: medium
status: inbox
tags: [spring]
---

# Spring MVC

## Definition
Spring's web framework built on the front-controller pattern: a single `DispatcherServlet` routes HTTP requests to handler methods.

## Why it matters
It's how REST endpoints are served. Understanding the request flow explains where filters, interceptors, validation, and exception handling fit.

## How it works
```text
Request -> DispatcherServlet -> HandlerMapping (find controller method)
   -> HandlerAdapter -> [Interceptors preHandle] -> Controller
   -> argument resolvers (@RequestBody, @PathVariable...)
   -> return value -> HttpMessageConverter (e.g. Jackson) -> Response
```
- `@RestController` = `@Controller` + `@ResponseBody`.
- Message converters serialize/deserialize (JSON via Jackson).

## Production usage
Keep controllers thin: validate input, delegate to services, map to DTOs. Centralize errors with `@ControllerAdvice` ([[Exception-Handling]]) and validate with [[Validation]].

## Trade-offs
- Servlet (blocking) model is simple and dominant; WebFlux (reactive) suits high-concurrency streaming but adds complexity.

## Common mistakes
- Business logic in controllers.
- Returning entities directly instead of DTOs (leaks schema, risks lazy-loading issues — see [[Hibernate-N-Plus-One]]).

## Interview questions
- Walk through the DispatcherServlet request lifecycle.
- Filter vs interceptor?

## Related concepts
- [[REST-Controller]]
- [[Exception-Handling]]
- [[Validation]]
