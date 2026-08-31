---
inclusion: auto
description: Parameterized queries, input validation, secrets management, secure defaults
---

# Secure Coding

- Use parameterized queries for ALL database access. Never concatenate SQL.
- Validate all external input at the boundary: type, length, format, range.
- Never hardcode secrets, tokens, or credentials in source code.
- Store secrets in environment variables or a secret manager — never in git.
- Use allow-lists over deny-lists for input validation.
- Escape output for its context (HTML, shell, LDAP, XML).
- Enforce authorization server-side on every protected operation.
- Fail closed: on error, deny by default.
- Never log PII, tokens, or stack traces to user-facing responses.
- Pin dependency versions. Prefer well-known, maintained packages.
- When creating any endpoint, explicitly state its auth posture.
- Set file permissions to 600 on any credential file.
- Use TLS in transit; encrypt sensitive data at rest.
