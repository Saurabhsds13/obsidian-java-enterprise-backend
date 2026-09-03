---
type: concept
domain: backend
topic: authentication
difficulty: medium
status: inbox
tags: [backend]
---

# Authentication

## Definition
Verifying **who** a caller is — establishing identity via credentials, tokens, or federated providers.

## Why it matters
It's the gate before [[Authorization]]. Weak authentication is a top security risk.

## How it works
- **Session-based**: server stores a session; client holds a session cookie. Good for browser apps.
- **Token-based ([[JWT]])**: stateless; client sends a bearer token per request.
- **OAuth2 / OpenID Connect**: delegate identity to a provider; the app trusts issued tokens.

## Production usage
Hash passwords with BCrypt/Argon2 (never plaintext, never reversible encryption). Use short-lived access tokens + refresh tokens. Enforce MFA for sensitive actions. Store secrets in a secret manager (`YOUR_JWT_SECRET`).

## Trade-offs
- Sessions: easy revocation, but stateful/harder to scale.
- Tokens: stateless/scalable, but revocation is hard (keep short-lived).

## Common mistakes
- Storing passwords reversibly.
- Long-lived tokens with no rotation.

## Interview questions
- Session vs token authentication?
- How do you handle token revocation?

## Related concepts
- [[Authorization]]
- [[JWT]]
- [[Spring-Security]]
