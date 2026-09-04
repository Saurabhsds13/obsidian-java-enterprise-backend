---
type: concept
domain: spring
topic: security
difficulty: hard
status: inbox
tags: [spring, backend]
---

# JWT

## Definition
JSON Web Token — a compact, URL-safe, **signed** token carrying claims (subject, roles, expiry). Structure: `header.payload.signature`, each Base64URL-encoded. It lets a server verify identity/claims **statelessly**, without a server-side session lookup.

## Why it matters
JWTs enable stateless auth across services (microservices, mobile, SPAs). Their strength (self-contained, no lookup) is also their weakness (hard to revoke, readable payload) — the trade-offs are prime interview territory and a common source of security bugs.

## How it works — the mechanism
- **header**: `{ "alg": "RS256", "typ": "JWT" }` — the signing algorithm.
- **payload**: claims — `sub`, `exp`, `iat`, `iss`, `aud`, plus custom (`roles`). **Base64, not encrypted** → readable by anyone. Never put secrets in it.
- **signature**: `sign(base64(header) + "." + base64(payload), key)`. The server recomputes and compares to verify integrity + authenticity.
- Sent as `Authorization: Bearer <token>`; the server validates signature and claims on every request — no DB hit.

### Signing: symmetric vs asymmetric
| | HMAC (HS256) | RSA/EC (RS256/ES256) |
|--|--------------|----------------------|
| Key | one shared secret | private (sign) + public (verify) |
| Who can verify | anyone with the secret (also can forge) | anyone with the public key (can't forge) |
| Best for | single service | multi-service: IdP signs, resource servers verify with public key |

Use **asymmetric** in microservices so resource servers verify without holding a forging key.

## The revocation problem (the key interview point)
A JWT is valid until `exp` — you can't "delete" it. Mitigations:
- **Short-lived access tokens** (minutes) + a long-lived **refresh token** (revocable, stored server-side).
- A **denylist** of revoked token ids (`jti`) in [[Redis]] until they expire (reintroduces some state — a pragmatic trade).
- Rotate signing keys (`kid` header) to invalidate en masse in an emergency.

## Validation checklist (must do all)
Verify the **signature**; check **`exp`** (not expired), **`iss`** (trusted issuer), **`aud`** (intended for us); pin the **algorithm** (reject `alg: none` and don't let the token pick a weaker alg — a classic attack).

## Enterprise example
```java
// Resource server verifies IdP-signed JWT via the IdP's public JWKS endpoint
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://idp.example.com/   # fetches public keys, validates iss/exp/signature
```

## Trade-offs
- Stateless + scalable + cross-service, but **hard to revoke**, larger than a session id, and payload is public. Keep them short-lived; store nothing sensitive.

## Common mistakes (senior-level)
- Long-lived tokens with no revocation path.
- Trusting claims without verifying the signature / accepting `alg: none`.
- Putting sensitive data (PII, secrets) in the payload (it's readable).
- Using the same symmetric secret across many services (any can forge).
- Not validating `iss`/`aud` → token from another system accepted.

## Interview questions (staff+)
- Structure of a JWT and how verification works.
- Why is revocation hard, and how do you handle it (short TTL + refresh + denylist)?
- Symmetric vs asymmetric signing — which for microservices and why?
- What's the `alg: none` attack and how do you prevent it?

## Related concepts
- [[Spring-Security]]
- [[Authentication]]
- [[Authorization]]
- [[API-Security]]
