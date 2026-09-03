---
type: concept
domain: java
topic: concurrency
difficulty: hard
status: inbox
tags: [java]
---

# CompletableFuture

## Definition
A `Future` that is also a `CompletionStage`: it can be completed manually and, more importantly, **composed** — you attach continuations (`thenApply`, `thenCompose`, `thenCombine`) that run when the stage completes, enabling non-blocking async pipelines and fan-out/fan-in orchestration.

## Why it matters
It's how you parallelize independent downstream calls without blocking threads, and how you attach timeouts/fallbacks declaratively. Interviewers probe the `thenApply` vs `thenCompose` distinction, the default-executor trap, and exception propagation — all easy to get subtly wrong.

## How it works — the mechanism
- Each stage holds a result-or-exception and a stack of dependent actions. Completing a stage triggers its dependents.
- **Which thread runs the continuation?**
  - `thenApply` (no executor) → runs on the thread that completed the previous stage, or the caller if already complete.
  - `thenApplyAsync` (no executor arg) → runs on the **common ForkJoinPool** (`ForkJoinPool.commonPool()`), sized to `cores - 1` — *shared process-wide*.
  - `thenApplyAsync(fn, myExecutor)` → runs on **your** executor. **Always pass your own executor for blocking work.**

### Composition operators
| Operator | Purpose | Analogy |
|----------|---------|---------|
| `thenApply` | transform result `T -> U` | `map` |
| `thenCompose` | chain another CF (avoid `CF<CF<U>>`) | `flatMap` |
| `thenCombine` | join two independent CFs | `zip` |
| `allOf` / `anyOf` | fan-in on many | barrier / race |
| `exceptionally` / `handle` / `whenComplete` | error handling | `catch` / `finally` |

## Enterprise example — fan-out with timeout + fallback
```java
CompletableFuture<Price> price = CompletableFuture.supplyAsync(() -> pricing.get(id), ioPool);
CompletableFuture<Stock> stock = CompletableFuture.supplyAsync(() -> inventory.get(id), ioPool);

Quote quote = price
    .thenCombine(stock, Quote::of)                 // join two parallel calls
    .orTimeout(300, TimeUnit.MILLISECONDS)         // Java 9+: fail if slow
    .exceptionally(ex -> Quote.unavailable(id))    // fallback, never leak the exception
    .join();                                        // block once, at the edge
```
Design rule: do async work in the middle, **block exactly once at the boundary** (`join`), and always terminate the chain with `exceptionally`/`handle`.

## thenApply vs thenCompose (the classic question)
```java
CompletableFuture<User>  u  = fetchUser(id);
// WRONG: nested future
CompletableFuture<CompletableFuture<Account>> bad = u.thenApply(user -> fetchAccount(user));
// RIGHT: flatten
CompletableFuture<Account> good = u.thenCompose(user -> fetchAccount(user));
```
Use `thenApply` when the function returns a **value**, `thenCompose` when it returns **another CompletableFuture**.

## Trade-offs
- Advantages: non-blocking composition, structured timeouts/fallbacks, parallel fan-out.
- Disadvantages: error handling and threading model are subtle; stack traces are async and harder to read; overuse where a simple sequential call would do adds cognitive load. For deep reactive needs, Reactor/`Mono` may fit better.

## Common mistakes (senior-level)
- Running blocking calls on the **common pool** (`*Async` without an executor) → starves every other user of that shared pool.
- Forgetting `exceptionally`/`handle` → failures vanish into an unobserved future.
- `thenApply` where `thenCompose` is needed → nested `CF<CF<T>>`.
- Calling `.get()`/`.join()` in the middle of a pipeline, re-introducing blocking.
- Ignoring cancellation semantics (`cancel` doesn't interrupt the running supplier).

## Interview questions (staff+)
- `thenApply` vs `thenApplyAsync` vs `thenApplyAsync(fn, executor)` — which thread runs, and which is safe for blocking work?
- `thenApply` vs `thenCompose`?
- How do you add a timeout and a fallback, and where should you block?
- Why is the common ForkJoinPool a footgun here?
- How does exception propagation differ across `exceptionally`, `handle`, `whenComplete`?

## Related concepts
- [[ExecutorService]]
- [[Thread]]
- [[Timeout]]
- [[Circuit-Breaker]]
