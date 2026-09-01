# Contributing — building on WGU Agent Foundation

This Power is the **template** for WGU's agent Powers. Follow these conventions so
every new Power (and every addition to this one) stays consistent and predictable.

## Conventions (non-negotiable)

- **Naming:** `wgu-` prefix, lowercase-with-hyphens (`wgu-agent-foundation`, `wgu-<domain>-power`).
- **Versioning:** semver in `plugin.json` `version`, from day one. Bump it on every meaningful change
  so consumers can pin a known-good baseline.
- **Layout (root-level, matches the Kiro Agent Plugins install shape):**
  ```
  <power>/
  ├── README.md        # what it is, how to install, what's inside (with a data-flow diagram)
  ├── plugin.json      # manifest: name, version, description, keywords, author, license
  ├── mcp.json         # MCP server config (governed — pointed at the gateway)
  ├── steering/        # guardrails + WGU context (.md rules)
  ├── skills/          # bundled skills
  └── docs/            # onboarding + this contributing guide
  ```

## Closed schemas — do NOT add fields

`plugin.json` and `mcp.json` follow the Agent Plugins **closed** schemas. Adding fields the
schema doesn't define gets them **ignored or rejected** by Kiro. Specifically:

- **`plugin.json`** — only the documented fields: `$schema`, `name`, `version`, `description`,
  `author`, `homepage`, `repository`, `keywords`, `license`. **No** `icon`, `oauth`, or `_comment`.
- **`mcp.json` remote server entry** — only `type`, `url`, `headers`. **No** `oauth` block
  (Kiro runs OAuth automatically on a 401), **no** `_comment`.

## How to add a new MCP server

1. Confirm the server should be **gateway-routed** (it almost always should — that's the governance point).
   Point its `url` at the WGU gateway door, not the vendor directly:
   ```json
   {
     "mcpServers": {
       "wgu-<tool>": {
         "type": "streamable-http",
         "url": "http://localhost:4000/mcp/<tool>"
       }
     }
   }
   ```
   (Local dev uses `localhost:4000`; a real deployment uses the gateway's ingress host.)
2. Add the matching **gateway route + backend + auth policy** on the gateway side — see the
   elicitation apply kit in `WGU-edu/wgu-devex-platform` (`deploy/gateway-auth/kit/`). The Power
   only *points* at the door; the gateway config is what *governs* the tool.
3. Add a few natural-language **activation keywords** to `plugin.json` `keywords` so the Power
   activates when a user mentions that tool (e.g. `confluence`, `page`, `space`).
4. Bump `plugin.json` `version`.

## How to add a new skill

1. Create `skills/<skill-name>/SKILL.md`. Keep it focused on one capability (e.g. "adversarial
   review", "PR flow").
2. Reference any supporting rules under `steering/` rather than duplicating them (DRY).
3. If the skill ships helper files (scripts, references), put them alongside `SKILL.md` in the
   skill's folder.
4. Bump `plugin.json` `version`.

## How to add / update a guardrail

1. Guardrails live in `steering/` as `.md` rules. One coherent concern per file.
2. If a guardrail is a WGU-wide standard, prefer vendoring it from the canonical source in
   `WGU-edu/wgu-devex-platform` (note the source in a comment) rather than forking it here.
3. Remember: guardrails **guide** (client-side). Anything that must be **guaranteed** belongs on
   the **gateway** (server-side policy), not in a steering file.

## Before you publish

- The Power repo must be **public** (Kiro imports by anonymous URL fetch; Internal/private 404s).
- **No secrets** in the repo — guardrails + gateway URLs only. Tokens/identity are the gateway's job.
- Validate the JSON against the closed schemas; confirm the Power activates and reads its skills in Kiro.
