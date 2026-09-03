---
type: concept
domain: spring
topic: spring-boot
difficulty: hard
status: inbox
tags: [spring]
---

# Spring Boot Auto-Configuration

## Definition
A mechanism that automatically configures beans based on the classpath, existing beans, and properties, so apps run with minimal explicit configuration.

## Why it matters
It's the core of Boot's "just works" experience and a common interview topic. Knowing how it decides prevents "why is this bean here?" confusion.

## How it works
- `@SpringBootApplication` includes `@EnableAutoConfiguration`.
- Boot reads auto-configuration classes listed in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 2.7+; previously `spring.factories`).
- Each is guarded by **conditions**: `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`, etc.
- Your beans win: `@ConditionalOnMissingBean` means defining your own bean disables the default.

```text
Add spring-boot-starter-data-jpa
  -> DataSource on classpath + spring.datasource.* present
  -> DataSourceAutoConfiguration creates a DataSource (unless you defined one)
```

## Production usage
Override defaults by declaring your own bean or setting properties. Debug with `--debug` to print the **conditions evaluation report** (matched/unmatched auto-configs).

## Trade-offs
- Advantages: fast setup, sensible defaults.
- Disadvantages: "magic" can hide behavior; classpath changes silently alter configuration.

## Common mistakes
- Not knowing a bean came from auto-config and double-defining it.
- Excluding the wrong auto-config class.

## Interview questions
- Explain Spring Boot auto-configuration. (see [[Explain-Spring-Boot-auto-configuration]])
- How do you override an auto-configured bean?

## Related concepts
- [[ApplicationContext]]
- [[Actuator]]
- [[IoC]]
