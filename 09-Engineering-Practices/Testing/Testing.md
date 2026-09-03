---
type: concept
domain: engineering-practices
topic: testing
difficulty: medium
status: inbox
tags: [engineering-practices]
---

# Testing

## Definition
Automated verification that code behaves as intended, across unit, integration, and contract levels.

## Why it matters
Tests enable safe refactoring, catch regressions early, and document behavior. They're expected in professional backends.

## The pyramid
- **Unit** (many, fast): a class/method in isolation; mock collaborators.
- **Integration** (fewer): components together — repository + real DB via **Testcontainers**, web layer via `@SpringBootTest`/`MockMvc`.
- **Contract** (targeted): verify producer/consumer API agreements between services.
- **End-to-end** (few): full flows; slow and brittle — keep minimal.

## Example
```java
@Test
void transfer_movesFunds() {
    accountService.transfer(from, to, TEN);
    assertThat(accountService.balance(from)).isEqualByComparingTo(NINETY);
}
```

## Production usage
Test business logic and edge cases; use Testcontainers for realistic DB tests; run in CI on every PR. Aim for meaningful coverage, not a number.

## Trade-offs
- Too many slow integration/E2E tests slow the pipeline; too few miss real bugs. Balance via the pyramid.

## Common mistakes
- Testing framework/getters instead of behavior.
- Flaky tests eroding trust.

## Interview questions
- Explain the test pyramid.
- Unit vs integration vs contract testing?

## Related concepts
- [[Clean-Code]]
- [[SOLID]]
