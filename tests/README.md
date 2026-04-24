# Tests

This directory contains a **manual QA checklist** for the Realize plugin. There is no automated harness — these scenarios are meant to be run by a human against a real Realize test account to verify end-to-end behavior.

## Prerequisites

1. Claude Code installed (`claude --version` works).
2. A Taboola Realize account (ideally a **test** or sandbox account — see Gotchas).
3. This plugin installed or loaded locally (see root [README](../README.md)).
4. Network access to `https://mcp.realize.com/mcp`.

## How to run

1. Open a Claude Code session in a directory where this plugin is active.
2. Open [`test-scenarios.md`](./test-scenarios.md).
3. For each scenario, type the **User prompt** into Claude Code and verify the **Expected behavior** matches.
4. Check off each scenario as it passes. If one fails, file an issue with:
   - Scenario number and title
   - Actual vs. expected behavior
   - Any error output
   - Claude Code version (`claude --version`)

## Gotchas

- **Use a test account.** Running report queries against a production account may hit rate limits or show real spend data to anyone observing your screen. As upstream adds tools that can mutate state, this becomes even more important — always run scenarios against a sandbox or dedicated test account, never production.
- **OAuth on first run.** Scenario 1 triggers an interactive browser-based OAuth flow. Have your Taboola SSO credentials ready.
- **Date windows.** Scenarios that reference relative dates ("last week") depend on your test account having data in that window. Adjust dates if your test account is empty.
- **CSV report truncation.** Very large result sets may be truncated server-side. If a report unexpectedly returns fewer records than `Total` would suggest, narrow the query.
