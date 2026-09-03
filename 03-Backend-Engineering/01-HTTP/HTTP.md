---
type: concept
domain: backend
topic: http
difficulty: easy
status: inbox
tags: [backend]
---

# HTTP

## Definition
The application-layer request/response protocol underlying web APIs. Clients send methods (GET, POST, ...) to URLs with headers and an optional body; servers respond with a status code, headers, and body.

## Why it matters
Every REST API is HTTP. Correct use of methods, status codes, and caching headers is the foundation of a good [[REST]] contract.

## How it works
- **Methods**: GET (read, safe), POST (create/act), PUT (replace), PATCH (partial update), DELETE.
- **Status classes**: 2xx success, 3xx redirect, 4xx client error, 5xx server error.
- **Caching**: `Cache-Control`, `ETag` + `If-None-Match` for conditional requests (304 Not Modified).
- **Versions**: HTTP/1.1 (keep-alive), HTTP/2 (multiplexing), HTTP/3 (QUIC/UDP).

```text
GET /api/v1/orders/42 HTTP/1.1
If-None-Match: "abc123"
--- 
HTTP/1.1 304 Not Modified
ETag: "abc123"
```

## Production usage
Use correct status codes ([[Pagination]] metadata, `201 Created` + `Location`), leverage ETags for caching, keep connections pooled/kept-alive.

## Trade-offs
- Statelessness scales horizontally but pushes state to tokens/DB/cache.

## Common mistakes
- 200 for everything (including errors).
- GET with side effects.

## Interview questions
- Idempotent vs safe methods?
- How do ETags reduce load?

## Related concepts
- [[REST]]
- [[Idempotency]]
- [[Caching]]
