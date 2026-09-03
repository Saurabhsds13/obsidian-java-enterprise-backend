---
type: concept
domain: java
topic: concurrency
difficulty: medium
status: inbox
tags: [java]
---

# CompletableFuture

## Definition
A `Future` that supports composition, chaining, and combining of asynchronous computations without blocking.

## Why it matters
Enables non-blocking orchestration of multiple async calls (e.g. fan-out to several services) and cleaner error handling than raw `Future`.

## How it works
```java
CompletableFuture<Price> price = CompletableFuture.supplyAsync(() -> priceService.get(id), pool);
CompletableFuture<Stock> stock = CompletableFuture.supplyAsync(() -> stockService.get(id), pool);

CompletableFuture<Quote> quote = price.thenCombine(stock, Quote::of)
    .orTimeout(500, TimeUnit.MILLISECONDS)
    .exceptionally(ex -> Quote.unavailable());
```
- `thenApply`/`thenCompose` chain; `thenCombine` joins; `allOf`/`anyOf` fan-in.
- Always supply an explicit executor for blocking work rather than the common ForkJoinPool.

## Production usage
Parallelize independent downstream calls, apply timeouts and fallbacks (pairs with [[Timeout]] and [[Circuit-Breaker]]).

## Trade-offs
- Advantages: composable, non-blocking.
- Disadvantages: error handling and threading model are easy to get subtly wrong.

## Common mistakes
- Running blocking calls on the default common pool.
- Forgetting `exceptionally`/`handle`, so failures vanish.

## Interview questions
- `thenApply` vs `thenCompose`?
- How do you add a timeout and fallback?

## Related concepts
- [[ExecutorService]]
- [[Thread]]
- [[Timeout]]
