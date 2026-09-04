---
type: concept
domain: backend
topic: authentication
difficulty: hard
status: inbox
tags: [backend]
---

# Authentication

## Definition
Verifying **who** a caller is — establishing identity via credentials, tokens, or a federated identity provider. Distinct from [[Authorization]] (what they may do), which happens after.

## Why it matters
It's the gate to everything. Broken authentication (weak password storage, guessable sessions, unverified tokens) is a top breach cause. Architect depth: session vs token trade-offs, OAuth2/OIDC roles, and revocation.

## How it works — the mechanisms
| Model | State | How | Best for |
|-------|-------|-----|----------|
| **Session (cookie)** | server-side session store | server issues a session id in a cookie; validated each request | browser apps, easy revocation |
| **Token (JWT)** | stateless | signed token sent as `Authorization: Bearer` ([[JWT]]) | APIs, microservices, mobile |
| **OAuth2 / OIDC** | delegated | app trusts tokens issued by an Identity Provider | SSO, third-party login, federation |

### OAuth2 vs OIDC (commonly confused)
- **OAuth2** = *authorization* framework (delegated access via access tokens/scopes) — "let this app call the API on my behalf."
- **OpenID Connect (OIDC)** = an *authentication* layer on top of OAuth2, adding the **ID token** (a JWT proving *who* the user is). Use OIDC for login, OAuth2 for delegated API access.

### Password storage (non-negotiable)
Store a **slow, salted hash**: bcrypt / scrypt / Argon2 (Argon2id preferred). Never plaintext, never fast hashes (MD5/SHA-256 alone — brute-forceable), never reversible encryption. The per-password salt defeats rainbow tables; the work factor defeats GPU cracking.

## Enterprise example — stateless resource server (OIDC)
```java
@Bean
SecurityFilterChain api(HttpSecurity http) throws Exception {
    return http
        .authorizeHttpRequests(a -> a.anyRequest().authenticated())
        .oauth2ResourceServer(o -> o.jwt(Customizer.withDefaults()))  // validate IdP-issued JWT
        .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
        .build();
}
```

## Trade-offs
| | Session | Token (JWT) |
|--|---------|-------------|
| State | server store (or Redis) | none (self-contained) |
| Scale | store is a dependency | trivially horizontal |
| Revocation | easy (delete session) | **hard** (valid until expiry) → keep short-lived + refresh |
| Size | small id | larger token per request |

## Common mistakes (senior-level)
- Storing passwords reversibly or with a fast hash (no salt/work factor).
- Long-lived access tokens with no rotation/revocation strategy.
- Trusting a JWT without verifying signature, `exp`, `iss`, `aud` ([[JWT]]).
- Confusing OAuth2 (authz) with OIDC (authn) — using access tokens as proof of identity.
- Rolling your own crypto/auth instead of a vetted library/IdP.

## Interview questions (staff+)
- Session vs token authentication — trade-offs, especially revocation.
- OAuth2 vs OIDC — which authenticates, which authorizes?
- How should passwords be stored, and why bcrypt/Argon2 over SHA-256?
- How do you revoke a stateless token?

## Related concepts
- [[Authorization]]
- [[JWT]]
- [[API-Security]]
- [[Spring-Security]]
