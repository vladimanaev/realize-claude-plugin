# CLAUDE.md — Contributor Notes

Internal implementation notes for anyone (human or agent) editing this plugin. Not user-facing — the user-facing docs are in [README.md](README.md).

## Architecture at a glance

This is a thin Claude Code plugin that wraps the [Realize remote MCP](https://github.com/taboola/realize-mcp). The MCP does all the heavy lifting (auth, HTTP, response shaping); the plugin's job is to teach the model **how to use the MCP well**.

```
┌─────────────────────┐
│ User (Claude Code)  │
└──────────┬──────────┘
           │ natural language
           ▼
┌─────────────────────┐
│  realize-analyst    │  orchestrator agent (agents/realize-analyst.md)
│       agent         │  routes intent → right skill/tool
└──────────┬──────────┘
           │
           ├──► accounts skill        → search_accounts
           ├──► campaigns skill       → get_all_campaigns, get_campaign, ...
           ├──► reports skill         → 4 report tools (CSV output)
           └──► create-campaign skill → UI walkthrough (no MCP writes)
                     │
                     ▼
┌────────────────────────────────────────┐
│ Realize MCP (https://mcp.realize.com)  │
│  OAuth 2.1, 11 tools at current rev    │
└────────────────────────────────────────┘
```

## Key design decisions

### No hooks
This plugin does not use Claude Code hooks. The remote MCP handles token refresh at the transport layer, so adding hooks here would be overhead without benefit.

### No direct curl / no API client code
All Realize API access flows through MCP tools. Do not add Bash curl calls that hit Realize endpoints directly — that bypasses the MCP's rate limiting, auth handling, and safety guarantees.

### Use only tools that actually exist upstream
The plugin's agent and skills must never fabricate tool calls. When a user requests an action that the current upstream MCP does not expose (e.g., creating a campaign), the `create-campaign` skill takes over with a UI walkthrough. When upstream adds new tools (including write tools), update the agent's Tool Reference, wire the new tool into the most appropriate skill, and trim the `create-campaign` UI walkthrough for the steps that become automatable — in an explicit PR, not silently.

### CSV, not JSON
Report tools return CSV. The leading metadata line (`Records: N | Total: M | Page: X | Size: Y`) is the primary pagination signal. Skills must cite `Total` in their summaries.

### All IDs are opaque strings from the API
Users often type numeric IDs in natural language. The MCP expects opaque string identifiers (e.g., `advertiser_12345_prod` for `account_id`) returned by its own tools. Every skill and the agent must route account lookups through `search_accounts` first and pass returned IDs through verbatim — no coercion to numbers, no re-casing, no stripping. This applies to `account_id`, `campaign_id`, and `item_id` alike.

## How to add a new skill

1. Create `skills/<skill-name>/SKILL.md` with the YAML frontmatter template:
   ```yaml
   ---
   name: <skill-name>
   description: <when to use, what it does>
   allowed-tools: ["Read", "Bash", "AskUserQuestion"]
   ---
   ```
2. Document: when to use, workflow, gotchas, example prompts.
3. If the skill wraps MCP tools, list them with one-liner descriptions.
4. Update [README.md](README.md) "Available Skills" table.
5. Add at least one scenario to [tests/test-scenarios.md](tests/test-scenarios.md).
6. Update [CHANGELOG.md](CHANGELOG.md) under the next unreleased version.

## How to sync with upstream MCP changes

When [taboola/realize-mcp](https://github.com/taboola/realize-mcp) ships a new version:

1. Diff the tool list in its README against our [`realize-analyst` tool reference](agents/realize-analyst.md).
2. If a tool is added:
   - Add it to the agent's Tool Reference section.
   - Route it into an existing skill (or create a new one).
   - Add a test scenario.
3. If a tool is removed or renamed:
   - Update the agent and skills.
   - Bump the minor version in [`plugin.json`](.claude-plugin/plugin.json) and note the breaking change in [CHANGELOG.md](CHANGELOG.md).
4. If **write tools** are added upstream: **do not silently enable them.** Open an issue first; adding writes changes the plugin's safety posture and needs explicit review.

## Conventions

- Plugin name in `plugin.json`: lowercase, no hyphens (`realize`).
- Skill directory names: lowercase, hyphen-separated (`create-campaign`).
- Agent filenames: `<name>.md` matching the `name:` frontmatter.
- YAML frontmatter must be valid — CI validates this (`.github/workflows/validate.yml`).
- Never write user credentials or tokens to any tracked file. Local per-user state lives in `.claude/*.local.md` (gitignored).

## Repo layout

```
.
├── .claude-plugin/plugin.json     # plugin manifest
├── .mcp.json                      # remote Realize MCP wiring
├── .github/workflows/validate.yml # CI: JSON + YAML frontmatter checks
├── agents/                        # orchestrator(s)
├── skills/                        # domain skills
├── tests/                         # manual QA scenarios
└── <governance docs>              # README, CLAUDE, CHANGELOG, ...
```

## Open items

These are placeholders that should be updated by the repo maintainer before the first public release:

- **`SECURITY.md`** — replace `security@taboola.com` with the real disclosure address if it differs.
- **`plugin.json` `author`** — currently `"Taboola"`; specify a team or individual maintainer if desired.
- **`homepage` / `repository` URLs** — confirm the repo lives at `github.com/taboola/realize-claude-plugin` or update.
- **`CONTRIBUTING.md` contact line** — replace the `TODO: add team alias` comment with the real Taboola Realize team contact.
