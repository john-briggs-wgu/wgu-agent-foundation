# wgu-pilot-gateway

**WGU pilot bundle — gateway-routed variant.** One power that installs WGU's engineering
**guardrails** (governance knowledge) **plus** the **Jira and GitHub MCP tools pointed at the
WGU Solo Agent Gateway**. Tool traffic flows through one governed door — auth, policy,
rate-limit, observability — instead of each client calling vendors directly.

> **Guides vs guarantees:** the guardrails *guide* the agent (client-side knowledge); the
> gateway *guarantees* governance on tool traffic (server-side, in the request path).

---

## Install

### Option A — from this GitHub repo (if your Kiro Powers panel supports "Add from GitHub URL")

Paste this repo's URL into Kiro's Powers panel → Add Power → from GitHub:

```
https://github.com/john-briggs-wgu/wgu-pilot-gateway
```

> Note: URL-based install is a Kiro IDE Powers-panel capability. If your Kiro version does
> not offer it, use Option B.

### Option B — clone + import (works everywhere)

```bash
git clone https://github.com/john-briggs-wgu/wgu-pilot-gateway.git
kiro-cli mcp import --file wgu-pilot-gateway/mcp.json --scope global
# The guardrails skill: Powers panel → Add Custom Power → Local Directory → this folder.
```

---

## Prerequisite: the local Solo Agent Gateway must be running

The tool servers point at `http://localhost:4000/mcp/<tool>`. That requires the local Solo
Agent Gateway reachable on port 4000 via port-forward:

```bash
export DOCKER_HOST="unix://$HOME/.rd/docker.sock"
kubectl port-forward -n agentgateway-system svc/agentgateway-proxy 4000:80 --address 127.0.0.1
```

And the gateway must have `/mcp/github` and `/mcp/jira` routes registered (backends pointing
at the real vendor MCP servers).

---

## Authentication (read this — it's the honest part)

Routing tool calls **through** the gateway changes who performs OAuth:

- **Today (local):** the gateway *routes and logs* traffic, but does **not** itself perform
  the vendor OAuth. Authenticated calls to GitHub/Atlassian through the gateway require
  **Entra on-behalf-of (OBO)** at the gateway — **designed, not yet built.**
- **Working fallback for authenticated calls:** the non-gateway `wgu-jira` / `wgu-github`
  powers use the vendors' hosted OAuth directly (sign in once, no token in files).
- **Target:** gateway does Entra-OBO → engineer signs in once via Entra SSO → the gateway
  acts on-behalf-of them against every tool. One identity, centrally governed.

---

## Status

Version 0.1.0 — **local pilot / proof-of-concept.** Gateway routing proven (the gateway logs
MCP requests it routes); authenticated end-to-end vendor calls pending Entra-OBO. Not for
production until the gateway address (`gateway.wgu.edu`) and Entra-OBO are in place.
