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
The controls that protect an API's confidentiality, integrity, and availability: authentication, authorization, input validation, transport security, and abuse protection.

## Why it matters
APIs are the primary attack surface of modern backends. Most breaches trace back to broken access control, injection, or exposed secrets.

## How it works
Layered controls:
- **AuthN/AuthZ**: verify identity ([[Authentication]]) and enforce least privilege ([[Authorization]]).
- **Input validation**: reject malformed input at the boundary ([[Validation]]); use parameterized queries to prevent injection.
- **Transport**: TLS everywhere; HSTS.
- **Secure headers & CORS**: explicit allow-lists, not wildcards.
- **Abuse protection**: [[Rate-Limiting]], payload size limits.
- **Secrets**: never in code/logs; use a manager (`YOUR_API_KEY`).

## Production usage
Follow OWASP API Security guidance. Enable secret scanning and dependency vulnerability alerts. Log auth failures for detection without leaking sensitive detail.

## Trade-offs
- Stronger controls add friction; balance security with developer/client experience.

## Common mistakes
- Broken object-level authorization (IDOR).
- Wildcard CORS with credentials.
- Secrets committed to the repo.

## Interview questions
- Top API security risks and mitigations?
- How do you prevent injection and IDOR?

## Related concepts
- [[Authentication]]
- [[Authorization]]
- [[JWT]]
- [[Rate-Limiting]]
