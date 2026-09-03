---
type: concept
domain: spring
topic: security
difficulty: hard
status: inbox
tags: [spring]
---

# Spring Security

## Definition
A framework for authentication (who you are) and authorization (what you may do) in Spring applications, built around a chain of servlet filters.

## Why it matters
It's the standard way to secure Spring APIs. Misconfiguration is a top source of vulnerabilities.

## How it works
- The **SecurityFilterChain** intercepts requests before controllers.
- **Authentication** establishes the principal (form login, JWT, OAuth2). See [[Authentication]].
- **Authorization** enforces access rules by URL or method (`@PreAuthorize`). See [[Authorization]].

```java
@Bean
SecurityFilterChain api(HttpSecurity http) throws Exception {
    return http
        .csrf(csrf -> csrf.disable())                 // stateless APIs use tokens
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/v1/public/**").permitAll()
            .anyRequest().authenticated())
        .oauth2ResourceServer(oauth -> oauth.jwt(Customizer.withDefaults()))
        .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
        .build();
}
```

## Production usage
Stateless [[JWT]]/OAuth2 for APIs; encode passwords with BCrypt; enforce least privilege with roles/authorities; keep [[API-Security|secure headers]] and CORS explicit. Never store secrets in code (`YOUR_JWT_SECRET`).

## Trade-offs
- Flexible but complex; the filter chain order matters.

## Common mistakes
- Disabling CSRF without understanding (fine for stateless token APIs, dangerous for cookie/session apps).
- Storing plaintext passwords.

## Interview questions
- Walk through the security filter chain.
- Authentication vs authorization?

## Related concepts
- [[JWT]]
- [[Authentication]]
- [[Authorization]]
- [[Spring-AOP]]
