---
name: "wgu-pilot-gateway-guardrails"
description: "Apply WGU's enterprise engineering guardrails when writing, reviewing, or committing code on any WGU project, AND understand that tool calls (Jira, GitHub) route through the WGU Solo Agent Gateway. Use when generating code, designing services, handling data or secrets, setting up CI, or using the bundled Jira/GitHub tools."
license: "MIT"
compatibility: "Works with any Agent Plugins v1.0.0 conformant client (validated against Kiro)"
metadata:
  author: "WGU Enterprise Architecture — Developer Experience"
  version: "0.1.0"
---

# WGU Pilot — Gateway-Routed Bundle

## Overview

This bundle is the "WGU way in a box" for the gateway pilot. It combines **two layers of
governance**:

1. **Guardrails (this knowledge)** — WGU's immutable org-level engineering standards that
   *guide* the agent client-side. Rules, not suggestions.
2. **Gateway-routed tools (the bundled MCP servers)** — Jira and GitHub calls flow through
   the WGU Solo Agent Gateway, which *governs* tool traffic server-side (auth, policy,
   rate-limit, observability).

**Guides vs guarantees:** the guardrails guide the agent's own behavior; the gateway
guarantees governance on the tool traffic (it physically sits in the request path — nothing
bypasses it).

## Engineering guardrails — when to apply each rule

| You are… | Load |
|----------|------|
| Writing or reviewing ANY code | [Secure Coding](references/secure-coding.md) |
| Adding a feature, fixing a bug, refactoring | [TDD Mandatory](references/tdd-mandatory.md) |
| Setting up CI/scripts, handling failures | [No Workarounds](references/no-workarounds.md) |
| Starting new work, choosing a pattern | [Memory First](references/memory-first.md) |

## Non-negotiables (summary)

- Secrets never in code or logs — env, a secret manager, or gateway-managed identity only.
- Parameterized queries always; validate external input at the boundary.
- State every endpoint's auth posture; enforce authz server-side; fail closed.
- Failing test first, then minimal code. No production code without a test.
- Never hide a failure (no `continue-on-error`, no `|| true`, no fake-data stubs).
- Search shared knowledge before writing new code; record decisions as ADRs.

## About the gateway-routed tools

- The bundled Jira and GitHub MCP servers point at the WGU Solo Agent Gateway, not the
  vendors directly. Every tool call is routed, policy-checked, and logged at one door.
- **Auth:** full per-user authentication through the gateway is Entra on-behalf-of (OBO) —
  designed, not yet built. Until OBO is live, the gateway routes and observes traffic; the
  vendor OAuth model (see the non-gateway `wgu-jira`/`wgu-github` powers) is the working
  fallback for authenticated calls.
- **Local vs production:** local uses `http://localhost:4000/mcp/<tool>` via port-forward;
  production swaps to `https://gateway.wgu.edu/mcp/<tool>`.

## Best practices
- Treat the guardrails as the floor, not the ceiling — team rules may be stricter.
- If a rule blocks you, fix the code; don't route around the rule.
