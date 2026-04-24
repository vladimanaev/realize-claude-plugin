# Changelog

All notable changes to this plugin will be documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [SemVer](https://semver.org/).

## [Unreleased]

### Added
- New `optimize-campaign` skill — data-driven diagnosis and recommendation loop grounded in Taboola's [official performance optimization guide](https://www.taboola.com/help/en/articles/3878108-how-to-improve-campaign-performance). Covers the 500–1000 click threshold, $50/day minimum spend rule, metric-combination prescription rules (CTR × CVR × CPA), and concrete UI-action recommendations with exact console paths.

### Changed
- `create-campaign` skill rewritten with the exact setup flow from Taboola's [official setup guide](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign): correct navigation (`Campaigns → +New → Campaign`), 5-value Marketing Objective enum, 3-value Bid Strategy enum with budget minimums (10× CPA for Maximize Conversions; 5× daily / 150× monthly for Enhanced CPC and Fixed Bid), 100–200 clicks/day rule for non-conversion campaigns, 3–10 ads per campaign recommendation, 7–10 day learning phase, 24–48 hour review cycle, and explicit guidance against narrowing targeting at launch.
- Agent `realize-analyst`: added optimization-routing example and responsibility, and updated the create-campaign example to reflect the fuller setup flow.
- README: added `optimize-campaign` row to skills table and expanded natural-language examples with optimization and setup prompts.

## [0.1.0] — 2026-04-24

### Added
- Initial scaffold: `.claude-plugin/plugin.json`, `.mcp.json` wiring the remote Realize MCP at `https://mcp.realize.com/mcp` via `type: streamable-http`.
- `realize-analyst` orchestrator agent.
- Skills: `accounts`, `campaigns`, `reports` (with `references/report-fields.md` and `references/csv-examples.md`), `create-campaign` (UI walkthrough for actions not currently exposed as MCP tools).
- `tests/test-scenarios.md` — 11 manual QA scenarios.
- Governance docs: README, CLAUDE.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md.
- CI workflow: `.github/workflows/validate.yml` — JSON + YAML frontmatter validation.
