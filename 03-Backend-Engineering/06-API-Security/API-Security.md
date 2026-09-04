---
type: concept
domain: backend
topic: api-security
difficulty: hard
status: inbox
tags: [backend]
---

# API Security

## Definition
The layered controls protecting an API's confidentiality, integrity, and availability: authentication, authorization, input validation, transport security, abuse protection, and secret management. Framed by the **OWASP API Security Top 10**.

## Why it matters
APIs are the primary attack surface of modern backends. Most breaches trace to broken access control, injection, or exposed secrets — all preventable with disciplined, layered controls (defense in depth).

## How it works — the layers (OWASP-aligned)
| Risk | Control |
|------|---------|
| Broken object-level auth (IDOR) | server-side ownership checks ([[Authorization]]) |
| Broken authentication | strong [[Authentication]], short-lived [[JWT]], MFA |
| Injection (SQL/command) | **parameterized queries**, input [[Validation]], least-privilege DB user |
| Excessive data exposure | DTOs, field allow-lists (never dump entities) |
| Lack of rate limiting | [[Rate-Limiting]] / [[Rate-Limiting-Design]], payload size caps |
| Security misconfig | secure headers, least privilege, disable debug endpoints |
| Mass assignment | bind only allowed fields (explicit DTOs, not entity binding) |

### Transport + headers + CORS
- **TLS everywhere** + HSTS; no plaintext.
- Security headers: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Strict-Transport-Security`.
- **CORS**: explicit origin allow-list; **never** `Access-Control-Allow-Origin: *` together with credentials.

### Secrets
Never in code, config, or logs. Use a secret manager (Vault, cloud KMS), rotate regularly, use placeholders in the repo (`YOUR_API_KEY`). Enable secret scanning + push protection.

## Enterprise example — injection prevention
```java
// SAFE: parameterized — the input can never become SQL
jdbc.query("SELECT * FROM users WHERE email = ?", rowMapper, email);
// UNSAFE: string concatenation -> SQL injection
// jdbc.query("SELECT * FROM users WHERE email = '" + email + "'");
```

## Trade-offs
- Stronger controls add friction (dev + client); balance security posture with usability — but never trade away the non-negotiables (authz checks, parameterized queries, TLS, secret hygiene).

## Common mistakes (senior-level)
- IDOR / missing object-level authorization.
- String-concatenated queries → injection.
- Returning entities → excessive data exposure + mass assignment on writes.
- Wildcard CORS with credentials.
- Secrets committed to the repo or logged.
- Verbose error messages leaking stack traces / internal detail.

## Interview questions (staff+)
- Walk the OWASP API Top 10 risks you defend against and how.
- How do you prevent injection and IDOR concretely?
- CORS: why is `*` + credentials dangerous?
- How do you manage secrets across environments?
- Why are DTOs a security control (exposure + mass assignment)?

## Related concepts
- [[Authentication]]
- [[Authorization]]
- [[JWT]]
- [[Rate-Limiting]]
- [[Validation]]
