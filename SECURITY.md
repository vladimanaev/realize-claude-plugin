# Security Policy

## Reporting a vulnerability

If you believe you have found a security issue in this plugin, please report
it privately. **Do not** open a public GitHub issue for security reports.

- Email: `security@taboola.com` <!-- TODO: confirm the correct address before first public release -->
- Include: a description of the issue, reproduction steps, and any relevant
  context (plugin version, Claude Code version, whether remote or local MCP
  was in use).

You can expect an initial acknowledgment within a few business days.

## Scope

This repository contains:

- Plugin configuration (`.claude-plugin/plugin.json`, `.mcp.json`)
- Markdown skills and an orchestrator agent
- Documentation and test scenarios

It does **not** bundle the Realize MCP server itself. For vulnerabilities in
the MCP server, please report them to
[taboola/realize-mcp](https://github.com/taboola/realize-mcp) directly.

## Out of scope

- User-supplied content (prompts, saved `account_id`s) stored locally in
  `.claude/*.local.md`. These files are gitignored and contain only
  per-user preferences — no tokens are stored by this plugin.
- OAuth token handling, which is performed by the Claude Code MCP transport
  layer, not by this plugin.
