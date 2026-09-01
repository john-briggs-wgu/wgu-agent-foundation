# WGU Agent Foundation

**WGU's foundational agent Power for Kiro:** engineering guardrails, WGU context,
and a starter set of governed MCP servers (Atlassian, GitHub) + skills. This is
the **template every future WGU Power inherits from** — install it once and your
agent behaves the WGU way and reaches tools through the one governed door.

> **No PATs in files. Identity, not tokens.**

---

## How it works

```
WGU Agent Foundation — how it works

┌───────────────────────────────────────────────────────┐
│  Kiro (the engineer's AI agent)                                │
│                                                                │
│   ┌────────────────────────────────────────────────┐  │
│   │  skills/  ·GUIDES·  (client-side KNOWLEDGE)                │  │
│   │   • wgu-pilot-gateway-guardrails: secure coding, TDD,     │  │
│   │     no-workarounds, memory-first (+ references/ rules)   │  │
│   │   • (future) adversarial review, PR flow, …              │  │
│   └────────────────────────────────────────────────┘  │
│                                                                │
│   mcp.json ─ points every tool at the WGU door, NOT the vendor │
└──────────────────────────────────┬────────────────────────────┘
                                 │  (all MCP tool calls)
                                 ▼
╔═══════════════════════════════════════════════════════╗
║  WGU Solo Agent Gateway  ·GUARANTEES· (server-side governance)   ║
║                                                                  ║
║   1. Identity check   ── who is calling?  (Entra JWT, per call)  ║
║   2. Consent + broker ── gets a vendor token on the user's       ║
║                          behalf, stored per-user (STS)           ║
║   3. Route · log · rate-limit · policy  (one auditable door)      ║
╚═════════════════════════════════╤══════════════════════════╝
                                 │  (brokered, authenticated)
                    ┌────────────┴────────────┐
                    ▼                          ▼
            ┌───────────────┐          ┌───────────────┐
            │   Atlassian   │          │    GitHub      │
            │  (Jira MCP)   │          │  (GitHub MCP)  │
            └───────────────┘          └───────────────┘

  GUIDES     = shipped in the Power's skills/, run on the engineer's machine (knowledge)
  GUARANTEES = enforced by the gateway, server-side (identity, audit, policy)
```

**The core distinction (never blur these):**

- **Guardrails GUIDE** — they are client-side knowledge (shipped as a **skill** in `skills/`)
  that shapes how the agent writes, reviews, and commits. They coach; they do not enforce.
- **The gateway GUARANTEES** — it physically sits in the request path, so identity
  checks, per-user token brokering, rate limits, policy, and audit are enforced
  server-side. You cannot route around it.

---

## What's inside

| Path | What it is |
|---|---|
| `plugin.json` | Power manifest (name, version, description, activation keywords). |
| `mcp.json` | Governed MCP servers — Atlassian + GitHub, pointed at the WGU gateway door. |
| `skills/` | Bundled skills Kiro loads from the Power. The `wgu-pilot-gateway-guardrails` skill carries the WGU engineering guardrails (secure coding, TDD, no-workarounds, memory-first) plus its `references/` rule files. Future skills (adversarial review, PR flow) go here too. |
| `docs/` | Onboarding + the contributing guide for building the *next* WGU Power. |

> **Where do the guardrails live?** In `skills/wgu-pilot-gateway-guardrails/` — Kiro loads a
> Power's `skills/`, so guardrails ship as a skill (with `references/` for the detailed rules),
> not as a separate top-level directory. (Kiro's workspace-level `.kiro/steering/*.md` is a
> different, complementary mechanism for per-workspace rules — not part of an installed Power.)

---

## Install

In Kiro, import this Power from its public GitHub URL:

```
https://github.com/john-briggs-wgu/wgu-agent-foundation
```

> **Why a public repo?** Kiro imports Powers by fetching the repo **anonymously**.
> An Internal/private repo returns 404 to that fetch, so the Power must be public
> to be installable by URL. The canonical WGU source of truth (ADRs, deploy kit,
> governance docs) lives in the Internal `WGU-edu/wgu-devex-platform` repo; this
> public repo carries **only** guardrails + gateway URLs (no secrets).

### Prerequisite: the gateway must be reachable

The MCP servers point at `http://localhost:4000/mcp/...`. Start a port-forward to
the **gateway proxy** (not the UI) before using the tools:

```bash
kubectl port-forward -n agentgateway-system svc/agentgateway-proxy 4000:80 --address 127.0.0.1
```

(See the setup runbook and the elicitation apply kit in `WGU-edu/wgu-devex-platform`
for standing up the gateway.)

---

## Quick start — try these right now

Once installed (guardrails work immediately; tools need the gateway reachable):

- *"Review this code for security issues"* → secure-coding guardrails engage.
- *"Help me write tests for this function"* → TDD (failing test first).
- *"Create a Jira ticket for the login bug"* → governed Jira tools + WGU ticket standards.
- *"Open a PR for this change"* → governed GitHub tools + code-quality guardrails.

---

## What's real vs. mock vs. blocked (honest status)

- ✅ **Guardrails skill** — real and active the moment you install (client-side knowledge).
- ✅ **Gateway routing + logging** — real: every tool call flows through the WGU door and is logged.
- ✅ **Gateway identity check + brokered auth** — demonstrated locally end-to-end against an
  Entra **mock** (the mock auto-approves the login step; all other OAuth mechanics are real).
- ⏳ **Real per-user production identity** — needs the WGU Entra app registration (tracked as
  **TAG-3857**). The mock → real swap is a one-file config change.

See `WGU-edu/wgu-devex-platform/docs/adr/008-mcp-auth-approaches-comparison.md` for the
full comparison and honest demo inventory.

---

## Conventions (this is the template — lock these in)

- **Naming:** `wgu-` prefix, lowercase-with-hyphens.
- **Versioning:** semver from day one (`plugin.json` `version`) so downstream repos pin a known-good baseline.
- **Layout:** `plugin.json` + `mcp.json` + `skills/` + `docs/` at the repo root.
- **Adding to it:** see `docs/contributing.md` for how to add a new MCP server or skill so the
  next WGU Power inherits the pattern instead of reinventing it.

---

*Author: WGU Enterprise Architecture. Canonical source: `WGU-edu/wgu-devex-platform`.*
