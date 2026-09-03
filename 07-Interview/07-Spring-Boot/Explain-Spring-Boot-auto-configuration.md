---
type: interview
domain: spring
topic: spring-boot
difficulty: hard
status: inbox
tags: [interview, spring]
---

# Explain Spring Boot auto-configuration

## Question
> How does Spring Boot auto-configuration decide what to configure?

## Short answer
`@EnableAutoConfiguration` loads candidate configuration classes that are guarded by conditions (classpath, existing beans, properties); matching ones contribute beans, and your own beans override the defaults.

## Detailed answer
Boot reads auto-configuration classes from `META-INF/spring/...AutoConfiguration.imports`. Each is annotated with conditions like `@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`. At startup Boot evaluates them: if the condition matches, the beans are created. Because of `@ConditionalOnMissingBean`, defining your own bean disables the default. Run with `--debug` for the conditions evaluation report.

## Example
Adding `spring-boot-starter-data-jpa` + a datasource config triggers `DataSourceAutoConfiguration` to create a `DataSource` — unless you define one yourself.

## Production relevance
Override defaults via properties or your own beans; debug surprises with the conditions report; be aware classpath changes alter configuration.

## Common mistake
Double-defining a bean that auto-config already provides, or not knowing where a bean came from.

## Follow-up questions
- How do you exclude an auto-configuration?
- Difference between a starter and auto-configuration?

## Related concepts
- [[Spring-Boot-Auto-Configuration]]
- [[ApplicationContext]]
