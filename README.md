# Realize Claude Plugin

Query Taboola **Realize** campaigns and pull performance reports through natural language, straight from Claude Code. Powered by the [Realize remote MCP](https://github.com/taboola/realize-mcp).

> **Scope today:** account discovery, campaign inspection, and performance reports. For actions the MCP does not yet expose as tools (e.g., creating or editing a campaign), the plugin walks you through the Realize console and then verifies the result via MCP.

## Prerequisites

- [Claude Code](https://claude.ai/claude-code) CLI installed (`claude --version` works)
- A Taboola Realize account (you'll authenticate via Taboola SSO on first use)
- Network access to `https://mcp.realize.com/mcp`

## Quick Start (Remote — recommended)

The plugin ships with the remote Realize MCP pre-wired. No local setup required — OAuth 2.1 is handled automatically by Claude Code.

Choose the path that matches how you consume Claude Code plugins:

### Option A — Install from a marketplace

Once this plugin is published to a Claude Code plugin marketplace, install it with the standard plugin flow (see the [Claude Code plugin docs](https://code.claude.com/docs/en/plugins) for the current command shape for your version). Claude Code reads this repo's `.claude-plugin/plugin.json` and `.mcp.json` on install and wires everything up.

### Option B — Local development / custom loading

Clone the repo and load it as a local plugin via `--plugin-dir`:

```bash
git clone https://github.com/taboola/realize-claude-plugin
claude --plugin-dir ./realize-claude-plugin
```

This loads the skills, agent, and MCP wiring directly from the repo without requiring a marketplace install. Use this for iterating on skills, testing scenarios, or running the plugin inside a restricted/air-gapped environment.

On first tool call, Claude Code opens a browser for Taboola SSO login and returns you to the terminal once authenticated.

### Option C — Just the MCP, without this plugin's skills/agent

If you only want the remote Realize MCP and not the opinionated skills/agent this plugin provides:

```bash
claude mcp add --transport http --callback-port 3000 realize-mcp https://mcp.realize.com/mcp
```

## Authentication

- **OAuth 2.1 via Taboola SSO.** On first tool call, your default browser opens to the Taboola login page. Enter your credentials; Claude Code stores the resulting token securely.
- **No manual token management.** Token refresh is handled by the transport layer.
- **Multi-user safe.** The remote endpoint is stateless and per-user — no shared credentials.

## Advanced: Local stdio fallback

If you need to run Realize MCP locally (air-gapped dev, custom routing, etc.), install the Python package and configure client credentials:

```bash
pip install realize-mcp
```

Then add to `.mcp.json` (or your Claude Code MCP config):

```json
{
  "mcpServers": {
    "realize-mcp": {
      "command": "realize-mcp-server",
      "env": {
        "REALIZE_CLIENT_ID": "your_client_id",
        "REALIZE_CLIENT_SECRET": "your_client_secret"
      }
    }
  }
}
```

For self-hosted HTTP mode and full local deployment details, see the [realize-mcp repo](https://github.com/taboola/realize-mcp).

## Available Skills

| Skill | Purpose |
|---|---|
| [`accounts`](skills/accounts/SKILL.md) | Find Realize accounts and capture the `account_id` every other tool needs |
| [`campaigns`](skills/campaigns/SKILL.md) | List and inspect campaigns and their creatives |
| [`reports`](skills/reports/SKILL.md) | Pull the four Realize performance reports and interpret the CSV output |
| [`optimize-campaign`](skills/optimize-campaign/SKILL.md) | Diagnose underperforming campaigns against Taboola's official playbook (500–1000 clicks per item, $50/day minimum, 7–10 day learning phase) and prescribe concrete UI actions |
| [`create-campaign`](skills/create-campaign/SKILL.md) | Walk through the Realize console setup flow (exact Marketing Objective enum, Bid Strategy × budget minimums, learning-phase defaults) for actions not yet exposed as MCP tools, plus MCP verification afterward |

The plugin also ships one agent, [`realize-analyst`](agents/realize-analyst.md), which routes natural-language questions to the right skill/tool and summarizes results in prose.

## Natural-language examples

```
"List my Realize accounts."
"Show me the active campaigns for Acme."
"Which content drove the most spend in the last 7 days?"
"Break down spend by campaign for the last 30 days."
"Why is CPC up on campaign 12345 this week?"
"Show me the top 50 site/day rows for campaign 12345."

# optimization
"My campaign 12345 is burning budget with no conversions — what should I do?"
"Which items on campaign 67890 should I pause?"
"Which sites are wasting my spend this month?"
"My CPA doubled over the last two weeks. Help me diagnose."

# setup (UI walkthrough)
"Create a new Online Purchases campaign with a $25 CPA target."
"Walk me through launching a Leads campaign in the UK."
```

## Configuration Reference

### `.mcp.json` (shipped with the plugin)
Points Claude Code at the remote Realize MCP. Usually nothing to edit.

### `.claude/realize.local.md` (optional, gitignored)
Persist a default account between sessions:

```yaml
---
default_account_id: advertiser_12345_prod
default_account_name: Example Advertiser
---
```

The `accounts` skill offers to write this for you.

## Troubleshooting

**OAuth browser doesn't open.**
Ensure port `3000` is free (the MCP transport uses it for the OAuth callback). Close any lingering process and retry.

**`search_accounts` returns nothing.**
Confirm you logged in to the correct Taboola SSO realm. If your org uses a non-default tenant, contact your Taboola account rep.

**Reports come back with `Records: 0`.**
The query window genuinely had no data, or you queried a campaign that didn't run in that window. Widen the date range or re-verify the `campaign_id`.

**What format are `account_id`, `campaign_id`, and `item_id` in?**
All three are **opaque identifiers** returned by the API. `account_id` is a string (e.g., `advertiser_12345_prod`) returned exclusively by `search_accounts`. `campaign_id` and `item_id` come from campaign/item tools. Pass them to follow-up calls exactly as received — don't reformat or coerce to numbers.

**The plugin tried to create a campaign and failed.**
Claude should route write-intent requests to the `create-campaign` skill (UI walkthrough) whenever no MCP tool exists for the requested action. If Claude attempted a tool call that doesn't exist or isn't documented, that's a bug — please [file an issue](https://github.com/taboola/realize-claude-plugin/issues) with the transcript.

**CSV output was truncated.**
Very large result sets are auto-truncated server-side. Narrow the query (shorter date range, specific `campaign_id`, higher sort discrimination) and retry.

## License

[Apache 2.0](LICENSE).

## Security

Please report security issues privately — see [SECURITY.md](SECURITY.md).

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
