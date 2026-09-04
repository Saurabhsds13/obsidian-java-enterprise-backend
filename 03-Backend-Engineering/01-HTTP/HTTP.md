---
type: concept
domain: backend
topic: http
difficulty: medium
status: inbox
tags: [backend]
---

# HTTP

## Definition
The application-layer request/response protocol underlying web APIs: a client sends a method + URL + headers + optional body; the server responds with a status code + headers + body. Its semantics (methods, status classes, caching, conditional requests) define what a correct [[REST]] API looks like.

## Why it matters
Every REST API *is* HTTP. Correct method/status/caching usage is the difference between a clean, cache-friendly, retryable API and a brittle one. The version differences (1.1/2/3) also matter for latency and connection behavior.

## How it works — the mechanism

### Method semantics (safe / idempotent)
| Method | Safe (no effect) | Idempotent | Body |
|--------|------------------|------------|------|
| GET | yes | yes | no |
| HEAD | yes | yes | no |
| PUT | no | yes | yes |
| DELETE | no | yes | no |
| POST | no | **no** | yes |
| PATCH | no | not necessarily | yes |
"Safe" ⇒ no server state change; matters for caching, prefetching, and [[Retry|retryability]] ([[Idempotency]]).

### Status code classes
- **2xx** success (200 OK, 201 Created + `Location`, 204 No Content).
- **3xx** redirect (301 permanent/cacheable, 302 temporary, 304 Not Modified).
- **4xx** client error (400, 401 unauth'd, 403 forbidden, 404, 409 conflict, 422 unprocessable, 429 rate-limited).
- **5xx** server error (500, 502 bad gateway, 503 unavailable, 504 gateway timeout).

### Caching + conditional requests
- `Cache-Control` (max-age, no-store, private/public), `ETag` + `If-None-Match`, `Last-Modified` + `If-Modified-Since` → **304 Not Modified** saves bandwidth and backend work.
- Conditional writes: `If-Match` (optimistic concurrency at the HTTP layer).

### Version evolution
| Version | Key change | Impact |
|---------|-----------|--------|
| HTTP/1.1 | keep-alive, one request per connection at a time | head-of-line blocking → domain sharding hacks |
| HTTP/2 | multiplexing over one TCP conn, header compression (HPACK), server push | fewer connections, faster; still TCP HoL blocking |
| HTTP/3 | over QUIC (UDP) | removes TCP HoL blocking, faster connection setup, better on lossy networks |

## Enterprise example
```text
GET /api/v1/orders/42
If-None-Match: "v7"
---
HTTP/1.1 304 Not Modified        # no body re-sent; client uses its cache
ETag: "v7"
Cache-Control: private, max-age=30
```

## Trade-offs
- Statelessness scales horizontally but pushes state to tokens/DB/cache.
- 301 (cacheable) vs 302 (always hits server) trades caching against control (see [[URL-Shortener]]).

## Common mistakes (senior-level)
- 200 for everything, including errors (breaks clients, monitoring, retries).
- GET with side effects (breaks caching/prefetch safety).
- Ignoring `ETag`/conditional requests → unnecessary payloads and load.
- Wrong status codes (500 for client errors; 200 with an error body).
- Not using 201+`Location` for creation or 409 for conflicts.

## Interview questions (staff+)
- Safe vs idempotent methods — and why it matters for retries/caching.
- How do ETags + conditional requests reduce load?
- HTTP/1.1 vs 2 vs 3 — what problem does each solve?
- When 301 vs 302; when 401 vs 403; when 409 vs 422?

## Related concepts
- [[REST]]
- [[Idempotency]]
- [[Caching]]
- [[API-Versioning]]
