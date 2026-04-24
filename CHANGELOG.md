# Changelog

All notable changes to this plugin will be documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [0.1.0] — 2026-04-24

### Added
- Initial scaffold: `.claude-plugin/plugin.json`, `.mcp.json` wiring the remote Realize MCP at `https://mcp.realize.com/mcp` via `type: streamable-http`.
- `realize-analyst` orchestrator agent.
- Skills: `accounts`, `campaigns`, `reports` (with `references/report-fields.md` and `references/csv-examples.md`), `create-campaign` (UI walkthrough for actions not currently exposed as MCP tools).
- `tests/test-scenarios.md` — 11 manual QA scenarios.
- Governance docs: README, CLAUDE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md.
- CI workflow: `.github/workflows/validate.yml` — JSON + YAML frontmatter validation.
