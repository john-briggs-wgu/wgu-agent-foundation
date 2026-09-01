---
name: "wgu-pilot-gateway-guardrails"
description: "Apply WGU's enterprise engineering guardrails when writing, reviewing, or committing code on any WGU project, AND understand that tool calls (Jira, GitHub) route through the WGU Solo Agent Gateway. Use when generating code, designing services, handling data or secrets, setting up CI, or using the bundled Jira/GitHub tools."
license: "MIT"
compatibility: "Works with any Agent Plugins v1.0.0 conformant client (validated against Kiro)"
metadata:
  author: "WGU Enterprise Architecture"
  version: "0.1.4"
---

# WGU Pilot - Gateway-Routed Bundle

## Quick start - try these right now

After installing this power, try saying any of these to your agent:

- **"Create a Jira ticket for the login page bug"** - the agent uses the governed Jira tools + follows WGU standards for how tickets should be written.
- **"Review this code for security issues"** - the guardrails activate with WGU's secure coding standards (parameterized queries, input validation, no hardcoded secrets).
- **"Help me write tests for this function"** - TDD guardrails kick in: failing test first, then minimal code.
- **"Open a PR for this change"** - GitHub tools via the governed gateway + code-quality guardrails.

The power gives your agent two things at once: **the WGU engineering standards** (so it writes code the WGU way) **and governed tool access** (so Jira/GitHub calls go through one auditable door).

## Overview

This bundle is the "WGU way in a box" for the gateway pilot. It combines **two layers of governance**:

1. **Guardrails (this knowledge)** - WGU's immutable org-level engineering standards that *guide* the agent client-side. Rules, not suggestions.
2. **Gateway-routed tools (the bundled MCP servers)** - Jira and GitHub calls flow through the WGU Solo Agent Gateway, which *governs* tool traffic server-side (auth, policy, rate-limit, observability).

**Guides vs guarantees:** the guardrails guide the agent's own behavior; the gateway guarantees governance on the tool traffic (it physically sits in the request path - nothing bypasses it).

## Engineering guardrails - when to apply each rule

| You are... | Load |
|----------|------|
| Writing or reviewing ANY code | [Secure Coding](references/secure-coding.md) |
| Adding a feature, fixing a bug, refactoring | [TDD Mandatory](references/tdd-mandatory.md) |
| Setting up CI/scripts, handling failures | [No Workarounds](references/no-workarounds.md) |
| Starting new work, choosing a pattern | [Memory First](references/memory-first.md) |

## Non-negotiables (summary)

- Secrets never in code or logs - env, a secret manager, or gateway-managed identity only.
- Parameterized queries always; validate external input at the boundary.
- State every endpoint's auth posture; enforce authz server-side; fail closed.
- Failing test first, then minimal code. No production code without a test.
- Never hide a failure (no `continue-on-error`, no `|| true`, no fake-data stubs).
- Search shared knowledge before writing new code; record decisions as ADRs.

## About the gateway-routed tools

- The bundled Jira and GitHub MCP servers point at the WGU Solo Agent Gateway, not the vendors directly. Every tool call is routed, policy-checked, and logged at one door.
- **Auth:** full per-user authentication through the gateway uses Entra on-behalf-of (OBO) / ID-JAG Cross-App Access. Until the WGU Entra app registration is complete, the gateway routes and observes traffic; the vendor OAuth model is the working fallback.
- **Local vs production:** local uses `http://localhost:4000/mcp/<tool>` via port-forward; production swaps to `https://gateway.wgu.edu/mcp/<tool>`.

## Best practices
- Treat the guardrails as the floor, not the ceiling - team rules may be stricter.
- If a rule blocks you, fix the code; don't route around the rule.
