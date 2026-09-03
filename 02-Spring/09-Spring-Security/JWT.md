---
type: concept
domain: spring
topic: security
difficulty: medium
status: inbox
tags: [spring, backend]
---

# JWT

## Definition
JSON Web Token — a signed, self-contained token carrying claims (subject, roles, expiry) that a server can verify without server-side session state.

## Why it matters
JWTs enable stateless authentication across services, a common choice for REST APIs and microservices.

## How it works
- Structure: `header.payload.signature`, each Base64URL-encoded.
- The server signs with a secret (HMAC) or private key (RSA/EC). Clients send it in `Authorization: Bearer <token>`. The server verifies the signature and expiry.
- Claims are **readable** (not encrypted) — never put secrets in the payload.

```text
Authorization: Bearer eyJhbGciOiJIUzI1Ni<...>.eyJzdWIiOiJ1c2Vy<...>.<signature>
```

## Production usage
Short-lived access tokens + refresh tokens; validate `exp`, `iss`, `aud`. Use asymmetric keys so resource servers verify without holding the signing secret. Store keys via a secret manager (`YOUR_JWT_SECRET`).

## Trade-offs
- Advantages: stateless, scalable, cross-service.
- Disadvantages: hard to revoke before expiry — keep them short-lived; larger than session IDs.

## Common mistakes
- Long-lived tokens with no revocation.
- Trusting claims without verifying the signature.
- Storing sensitive data in the payload.

## Interview questions
- How would you revoke a JWT? Why is it hard?
- Symmetric vs asymmetric signing?

## Related concepts
- [[Spring-Security]]
- [[Authentication]]
- [[Authorization]]
