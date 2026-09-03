---
type: concept
domain: spring
topic: spring-core
difficulty: hard
status: inbox
tags: [spring]
---

# Bean Lifecycle

## Definition
The ordered sequence the Spring container drives each bean through: definition loading, instantiation, dependency population, `*Aware` callbacks, `BeanPostProcessor` hooks (where **AOP proxies are created**), initialization callbacks, active use, and destruction. Understanding the exact order explains where proxies appear, why self-injection is tricky, and where to open/close resources.

## Why it matters
Half of "my `@Transactional`/`@Cacheable` doesn't work in `@PostConstruct`" and "circular dependency" problems come from not knowing *when* proxies are created and *when* dependencies are ready. It's the backbone connecting [[ApplicationContext]], [[Dependency-Injection]], and [[Spring-AOP]].

## How it works — the ordered mechanism
```
1. Load BeanDefinitions (scan / @Configuration)
2. BeanFactoryPostProcessors run  (e.g. property placeholder resolution) — operate on DEFINITIONS
   ↓  (container then instantiates singletons eagerly)
3. Instantiate (constructor) — constructor injection happens here
4. Populate properties (field/setter injection)
5. Aware callbacks: BeanNameAware, BeanFactoryAware, ApplicationContextAware
6. BeanPostProcessor.postProcessBeforeInitialization
7. Init callbacks: @PostConstruct  ->  InitializingBean.afterPropertiesSet()  ->  @Bean(initMethod)
8. BeanPostProcessor.postProcessAfterInitialization   <-- AOP PROXY CREATED HERE
   ↓  bean is now in use (as a proxy if advised)
9. On shutdown: @PreDestroy -> DisposableBean.destroy() -> @Bean(destroyMethod)
```
Two load-bearing facts:
- **Constructor injection runs before any init callback**, so dependencies are guaranteed non-null in `@PostConstruct` — but proxy-based behavior isn't active on *self* calls yet.
- **The proxy is created in step 8** (`postProcessAfterInitialization` by the auto-proxy `BeanPostProcessor`). So a `this`-call inside `@PostConstruct` hits the raw target — advice won't apply (ties to the self-invocation problem in [[Spring-AOP]]/[[Transactional]]).

## BeanFactoryPostProcessor vs BeanPostProcessor
| | BeanFactoryPostProcessor | BeanPostProcessor |
|--|--------------------------|-------------------|
| Operates on | bean **definitions** (metadata) | bean **instances** |
| Timing | before instantiation | around initialization |
| Example | `PropertySourcesPlaceholderConfigurer` | AOP auto-proxying, `@Autowired` resolution |

## Enterprise example
```java
@Component
public class ConnectionWarmer {
    private final DataSource ds;                 // injected via constructor (step 3)
    public ConnectionWarmer(DataSource ds) { this.ds = ds; }

    @PostConstruct                                // step 7: deps ready
    void warm() { try (var c = ds.getConnection()) { c.isValid(1); } catch (Exception e) { /* fail fast */ } }

    @PreDestroy                                   // step 9: graceful shutdown
    void close() { /* release pools, flush buffers */ }
}
```

## Scopes and lifecycle
- **Singleton** (default): created eagerly at startup, destroyed on context close (`@PreDestroy` runs).
- **Prototype**: created per request; Spring does **not** manage destruction — `@PreDestroy` won't fire (you must clean up).
- **request/session**: web scopes, backed by a proxy so a singleton can hold a scoped bean.

## Circular dependencies
- **Constructor–constructor cycle**: unresolvable → `BeanCurrentlyInCreationException` (and Boot 2.6+ disallows by default). Fix the design.
- **Setter/field cycle**: Spring resolves it with an early singleton reference exposed before full init — but if AOP is involved, the injected reference may be the raw object rather than the proxy. Prefer refactoring over `@Lazy` band-aids.

## Trade-offs
- Eager singleton init surfaces wiring/config errors at startup (fail fast) but adds startup latency; heavy `@PostConstruct` work delays readiness.

## Common mistakes (senior-level)
- Calling advised (`@Transactional`/`@Cacheable`) methods from `@PostConstruct` or via `this` (proxy not active on self-calls).
- Expecting `@PreDestroy` on prototype beans (never runs).
- Doing slow/network work in `@PostConstruct`, breaking liveness/readiness ([[Actuator]]).
- Masking a circular-dependency design smell with `@Lazy`.

## Interview questions (staff+)
- Give the lifecycle order and pinpoint where the AOP proxy is created.
- Why can't you use `@Transactional` self-calls in `@PostConstruct`?
- `BeanFactoryPostProcessor` vs `BeanPostProcessor`?
- How does Spring resolve setter vs constructor circular dependencies, and what breaks with AOP?
- Why doesn't `@PreDestroy` run for prototype beans?

## Related concepts
- [[ApplicationContext]]
- [[Dependency-Injection]]
- [[Spring-AOP]]
- [[Transactional]]
