---
type: concept
domain: engineering-practices
topic: quality
difficulty: easy
status: inbox
tags: [engineering-practices]
---

# Clean Code

## Definition
Code that is easy to read, understand, and change — favoring clarity over cleverness.

## Why it matters
Code is read far more than written. Readable code lowers defect rate and onboarding cost.

## Principles
- Meaningful names; small, single-purpose functions.
- **DRY** (don't repeat yourself), **KISS** (keep it simple), **YAGNI** (don't build for imagined futures).
- Few arguments; avoid deep nesting (early returns); express intent over comments.
- Errors handled deliberately; no dead code.

## Example
```java
// Intent-revealing over clever
boolean isEligibleForRefund(Order o) {
    return o.isPaid() && o.withinRefundWindow();
}
```

## Production usage
Enforce with code review, formatters/linters, and consistent conventions. Balance with pragmatism and deadlines.

## Trade-offs
- Over-refactoring for "cleanliness" can churn code and add abstraction nobody needs (YAGNI).

## Common mistakes
- Premature abstraction; comments that restate code; giant functions/classes.

## Interview questions
- DRY vs premature abstraction — where's the line?
- What makes a good function?

## Related concepts
- [[SOLID]]
- [[Testing]]
