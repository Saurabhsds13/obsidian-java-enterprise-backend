---
type: concept
domain: spring
topic: hibernate
difficulty: hard
status: inbox
tags: [spring]
---

# Hibernate N+1 Problem

## Definition
A query-amplification anti-pattern: fetching N parent entities triggers N *additional* lazy queries — one per parent — to load an association, for **1 + N** round-trips instead of 1 or 2. It is the single most common ORM performance defect in production.

## Why it matters
It turns an endpoint that looks like one query in code into hundreds of queries at runtime, silently, and only shows up under real data volume. Diagnosing and fixing it — with the right fix per situation — is a core [[Hibernate]]/[[JPA]] competency.

## How it works — the mechanism
```java
List<Order> orders = orderRepo.findAll();   // Query 1: SELECT * FROM orders  (N rows)
for (Order o : orders) {
    o.getCustomer().getName();              // each lazy proxy access -> its own SELECT
}                                            // Queries 2..N+1: SELECT * FROM customers WHERE id = ?
```
Each lazy association access initializes a proxy with a separate SELECT. Same happens iterating a lazy `@OneToMany`. Ironically, `FetchType.EAGER` doesn't fix it — Hibernate still often issues per-association selects for eager `@ManyToOne` when loading a list.

## The fixes (choose per access pattern)
| Technique | How | Best for |
|-----------|-----|----------|
| **Fetch join** (JPQL) | `select o from Order o join fetch o.customer` | a known, single-valued association on a specific query |
| **Entity graph** | `@EntityGraph(attributePaths = "customer")` on the repo method | declarative, reusable per query, no custom JPQL |
| **Batch fetching** | `@BatchSize(size = 50)` or `hibernate.default_batch_fetch_size` | loads lazy associations in `IN (...)` batches → 1 + ceil(N/size) queries |
| **DTO projection** | `select new com.x.OrderView(o.id, c.name) from Order o join o.customer c` | read/list endpoints — fetch exactly the columns, skip entities entirely |

### The cartesian-product trap
Fetch-joining **two collections** in one query multiplies rows (N×M) and can blow up memory. Rules:
- Fetch **at most one collection** per query; batch-fetch the rest.
- Use `DISTINCT` (or `hibernate.query.passDistinctThrough=false` for the modern behavior) to dedupe the entity result of a collection fetch join.
- For paginating a collection fetch join, Hibernate paginates **in memory** (dangerous) — paginate the root with batch fetching instead.

## Enterprise example — the good version
```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    @EntityGraph(attributePaths = {"customer"})            // no N+1 for customer
    List<Order> findByStatus(OrderStatus status);

    // list view: project straight to a DTO, no lazy anything
    @Query("select new com.acme.OrderView(o.id, c.name, o.total) " +
           "from Order o join o.customer c where o.status = :s")
    List<OrderView> findViews(@Param("s") OrderStatus status);
}
```

## Detection
- Turn on SQL logging in dev (`spring.jpa.show-sql`, or better, `logging.level.org.hibernate.SQL=DEBUG` + `datasource-proxy`/Hypersistence for query counts).
- Assert query counts in integration tests (fail the build if a repo call exceeds a threshold).
- In prod, statement metrics / APM spans show the repeated identical query signature.

## Trade-offs
- Fetch joins minimize round-trips but risk cartesian blowups and awkward pagination.
- Batch fetching is a great default (bounded queries) but still more than one round-trip.
- DTO projections are fastest for reads but bypass the entity model (no dirty checking) — perfect for queries, not for writes.

## Common mistakes (senior-level)
- "Fixing" N+1 by switching associations to `EAGER` globally → moves/worsens the problem and creates giant joins.
- Fetch-joining multiple collections → cartesian explosion.
- Paginating a collection fetch join (in-memory pagination warning).
- Not detecting it until production because tests use tiny datasets.

## Interview questions (staff+)
- What exactly causes N+1, and why doesn't EAGER fix it?
- Compare fetch join vs entity graph vs batch size vs DTO projection — when do you use each?
- Why can fetch-joining two collections be catastrophic, and how do you paginate safely?
- How would you detect N+1 automatically in CI?

## Related concepts
- [[Hibernate]]
- [[JPA]]
- [[Query-Optimization]]
- [[Pagination]]
