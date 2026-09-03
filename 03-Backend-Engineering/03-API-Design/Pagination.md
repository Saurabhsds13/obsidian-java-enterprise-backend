---
type: concept
domain: backend
topic: api-design
difficulty: medium
status: inbox
tags: [backend]
---

# Pagination

## Definition
Returning large result sets in bounded pages instead of all at once, using offset-based or cursor-based schemes.

## Why it matters
Unbounded list endpoints exhaust memory and slow the DB. Pagination protects both server and client.

## How it works
- **Offset/limit**: `?page=2&size=50` → `LIMIT 50 OFFSET 100`. Simple, but deep offsets are slow and can skip/duplicate rows when data changes.
- **Cursor (keyset)**: `?after=<lastId>` → `WHERE id > :lastId ORDER BY id LIMIT 50`. Stable and fast at any depth; needs a sortable, unique cursor.

```text
GET /api/v1/orders?size=50&after=eyJpZCI6MTAwfQ==
{
  "items": [ ... ],
  "nextCursor": "eyJpZCI6MTUwfQ=="
}
```

## Production usage
Prefer cursor pagination for large or frequently-changing datasets and infinite scroll. Back it with an [[Indexes|index]] on the sort key.

## Trade-offs
- Offset: easy, allows jumping to a page, but slow deep and unstable.
- Cursor: fast and stable, but no random page jumps.

## Common mistakes
- Deep offset pagination on large tables (full scan pain).
- Missing index on the sort column.

## Interview questions
- Offset vs cursor pagination trade-offs?
- Why does deep offset pagination get slow?

## Related concepts
- [[REST]]
- [[Indexes]]
- [[Query-Optimization]]
