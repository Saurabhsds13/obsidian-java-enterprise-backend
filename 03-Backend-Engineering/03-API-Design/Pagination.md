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
Returning large result sets in bounded pages instead of all at once, via **offset-based** (`page`/`size`) or **cursor/keyset-based** (`after=<token>`) schemes. It protects server memory, DB load, and client bandwidth.

## Why it matters
An unbounded list endpoint is a latent outage — one client requesting everything can OOM the service or table-scan the DB. Choosing the right scheme (and indexing it) is a common design/review point.

## How it works — the mechanism

### Offset vs keyset
| | Offset (`LIMIT n OFFSET m`) | Keyset / cursor (`WHERE id > :last`) |
|--|------------------------------|--------------------------------------|
| Cost at depth | O(offset) — scans + discards m rows | O(log n) — index seek, constant at any depth |
| Stability under writes | rows can shift → skip/duplicate | stable (anchored to a key) |
| Random page jump | yes (`page=57`) | no (only next/prev) |
| Total count | easy | expensive/omit |
| Index need | on sort col | on the sort/cursor col (mandatory) |

- **Offset** is simple and allows jumping to a page, but **deep offset** scans and throws away all preceding rows (slow) and can skip/duplicate rows when the underlying data changes mid-paging.
- **Keyset** seeks directly past the last-seen key using the index → fast and stable at any depth. The cursor is an opaque, encoded token of the sort key(s).

## Enterprise example — opaque cursor
```text
GET /api/v1/orders?size=50&after=eyJjcmVhdGVkQXQiOiIyMDI2LTAxLTAxIiwiaWQiOjEwMH0
{
  "items": [ ... 50 ... ],
  "nextCursor": "eyJjcmVhdGVkQXQiOiIyMDI2LTAxLTAyIiwiaWQiOjE1MH0",
  "hasMore": true
}
```
```sql
-- backed by an index on (created_at, id); tie-break on id to keep it deterministic
SELECT * FROM orders
WHERE (created_at, id) > (:lastCreatedAt, :lastId)
ORDER BY created_at, id LIMIT 50;
```

## Trade-offs
- Offset: easy + page jumps, but slow deep and unstable → fine for small/admin datasets.
- Keyset: fast + stable, but no arbitrary page jumps and needs a unique, ordered cursor (add a tie-breaker like `id` when the sort column isn't unique).

## Common mistakes (senior-level)
- Deep offset pagination on large tables (full scan of skipped rows).
- Keyset on a non-unique sort column without a tie-breaker → skipped/duplicated rows at boundaries.
- Missing index on the sort/cursor column → every page is a scan.
- No max `size` cap → a client requests `size=1000000`.
- Returning an expensive total count on every page.

## Interview questions (staff+)
- Offset vs keyset — why does deep offset get slow and unstable?
- How do you build a correct cursor when the sort column isn't unique?
- Which index makes keyset pagination O(log n)?
- How do you bound abuse (max page size)?

## Related concepts
- [[REST]]
- [[Indexes]]
- [[Query-Optimization]]
