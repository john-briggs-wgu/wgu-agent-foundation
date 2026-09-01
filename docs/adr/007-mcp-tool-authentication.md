# WGU DevEx — MCP Tool Authentication Architecture

**Status:** Architecture / decision brief
**Author:** John Briggs (DevEx EA)
**Date:** 2026-08-31
**Applies to:** wgu-jira, wgu-github, and all future tool powers that reach WGU-controlled systems

---

## Purpose

Define how engineers authenticate to MCP tool servers (Jira, GitHub, etc.) **without personal access tokens in files**, and distinguish two authentication models — per-tool user OAuth (available today) and centralized Entra on-behalf-of (the target). Guiding principle: **no long-lived secrets in any config file. Identity, not tokens.**

## The problem we are eliminating

The original tool powers used the PAT-in-env model (`"env": {"GITHUB_TOKEN": ""}`). Every engineer pastes a personal access token into config — long-lived plaintext creds, no central revocation, weak audit, token sprawl.

## Model A — Per-tool user OAuth (AVAILABLE TODAY)

Point the power at the vendor's hosted OAuth MCP server. No token in config; the engineer signs in via browser and the vendor manages the token.

| Power | Endpoint |
|---|---|
| wgu-github | `https://api.githubcopilot.com/mcp/` |
| wgu-jira | `https://mcp.atlassian.com/v1/mcp/authv2` (Rovo, OAuth 2.1 + DCR) |

```json
{ "mcpServers": { "wgu-github": { "type": "http", "url": "https://api.githubcopilot.com/mcp/", "oauth": {} } } }
```

**SSO:** inherited, not configured by the power. If WGU's GitHub Enterprise and Atlassian orgs are federated to Entra (enforced SAML/OIDC SSO), the vendor login redirects to Entra. You CANNOT pin the Entra path from mcp.json — no field selects an IdP; enforcement lives in the vendor tenant or the gateway.

## Model B — Centralized Entra on-behalf-of (TARGET / blue column)

Point every power at WGU's own Agent Gateway (`/mcp`). The gateway is the single OAuth client; it authenticates the engineer via Entra SSO once and acts on-behalf-of them against the vendor. One identity all tools, central revocation, full audit, enforceable Entra.

## PROVEN REFERENCE PATTERN — gateway-side Entra auth is BUILDABLE

A complete working reference exists at **github.com/zackwy/virtual-mcp** (Solo's Virtual-MCP demo, Solo Enterprise agentgateway v2026.8.0). It implements the dual-OAuth, Entra-authenticated, no-PATs model.

**Dual-OAuth flow:**
- **Downstream (who is the caller):** `EnterpriseAgentgatewayPolicy` with `traffic.jwtAuthentication` validates the Entra JWT on `/mcp` (issuer `sts.windows.net/<tenant>/`, audience `api://<client-id>`, JWKS via a static backend to `login.microsoftonline.com`) + RFC 9728 metadata. Works on v2026.7.0+.
- **Upstream (act on the user's behalf):** per-backend `entElicitation.brokered.chainedAuth.oauth` + `entTokenExchange.solo` runs a second OAuth flow against the vendor with a gateway-branded consent screen; token stored keyed to the user's Entra identity, auto-refreshed. Requires v2026.8.0+.
- **OAuth issuer proxy:** gateway exposes `/oauth-issuer` so the browser reaches the consent screen and both callbacks through the same gateway address; `agentgateway.dev/issuer-proxy` annotation routes login through the gateway.

**Critical per-vendor difference:**

| Vendor | Registration | What you must do |
|--------|--------------|------------------|
| Atlassian (Rovo) | Dynamic Client Registration — gateway self-registers | `baseUrl` MUST equal Atlassian's advertised issuer (verify with the well-known endpoint). Upstream path `/v1/mcp`. |
| GitHub | NO DCR | Hand-register a GitHub App (not OAuth App), static clientId+secret as a k8s secret, enable "Expire user authorization tokens" for refresh. |

**Bonus — per-identity tool RBAC:** the same policy supports CEL `matchExpressions` on `jwt.roles` (Entra app roles) giving each caller a different tool catalog from the same `/mcp` URL, enforced on tools/list and tools/call.

**Prerequisites:** (1) WGU Entra app registration (`ENTRA_TENANT_ID`/`ENTRA_CLIENT_ID`, `mcp_access` scope, app roles, Web redirect + client secret for the issuer); (2) gateway v2026.8.0+; (3) a hand-registered GitHub App for the GitHub upstream leg (Atlassian uses DCR).

**Proven vs pending:** pattern is real/public/version-verified; local lab upgraded to v2026.8.0 with `entElicitation` present; downstream JWT-auth manifests adapted. NOT yet run end-to-end — needs the WGU Entra app registration + GitHub App. Do NOT claim it working until a live sign-in is observed.

## Cross-team dependencies

| Need | Owner |
|---|---|
| GitHub + Atlassian SSO federation to Entra | Identity team |
| Gateway Entra-OBO app registration | Identity team |
| Downstream token acceptance + scopes | Identity + admins |
| Powers pointed at vendor OAuth (Model A) | DevEx — done |
| Powers pointed at gateway (Model B) | DevEx — after gateway OBO exists |

_Full version of this ADR (with the local-lab proof logs and routing details) lives in WGU-edu/wgu-devex-platform docs/adr/._
