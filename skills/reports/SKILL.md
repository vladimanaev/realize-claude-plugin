---
name: reports
description: Pull Realize performance reports (CSV) and interpret them. Covers top-content, breakdown, history, and site/day breakdown reports with sort, filter, and pagination rules.
allowed-tools: ["Read", "Bash", "AskUserQuestion"]
---

# Reports

Wraps the four Realize MCP report tools. Reports return **CSV**, not JSON — interpret the output in prose rather than dumping it back at the user.

## Prerequisites

- `account_id` resolved via the `accounts` skill.

## Tools this skill wraps

| Tool | Sort | Filters | Good for |
|---|---|---|---|
| `mcp__realize-mcp__get_top_campaign_content_report` | ✓ | — | "What content performed best?" |
| `mcp__realize-mcp__get_campaign_breakdown_report` | ✓ | ✓ | "Break down campaign X by <dimension>" |
| `mcp__realize-mcp__get_campaign_history_report` | — | — | "How did campaign X trend over time?" |
| `mcp__realize-mcp__get_campaign_site_day_breakdown_report` | ✓ | ✓ | "Which sites/days drove the results?" |

**Sort.** `sort_field` is one of `"clicks"`, `"spent"`, `"impressions"`. `sort_direction` is `"ASC"` or `"DESC"` (uppercase, default `"DESC"`).

**Filters.** Named `filters` (plural). The parameter is a **flat JSON object with string-only values** that is merged into the upstream API's query string as-is. Example: `{"campaign_status": "RUNNING", "region": "US"}`. There is no predefined filter schema — keys and values are forwarded verbatim to the Realize API, and unknown keys are passed through without error on the MCP side. Nested objects and arrays are not accepted.

**Dates.** `start_date` and `end_date` are required on every report tool. Format: `YYYY-MM-DD` (ISO date, no time component, no timezone).

## CSV output format

Every response starts with a report header and a **metadata line** prefixed with `📊`:

```
🏆 **<Report Name> CSV** - Account: <account_id> | Period: <start_date> to <end_date>

📊 Records: 250 | Total: 1500 | Page: 1 | Size: 250

<csv header row>
<csv data rows...>
```

Always cite `Total` in your summary so the user knows the query's scope. If the response includes a `⚠️ **TRUNCATED**` banner, surface it — the data you see is incomplete.

See `references/csv-examples.md` for concrete sample outputs and `references/report-fields.md` for what each report returns.

## Pagination rules

- **Defaults:** `page_size=20`, `page=1`.
- **Max `page_size`:** 100 (server raises `ToolInputError` if exceeded).
- **Keep `page_size` constant** across pages in a single query session — changing it mid-pagination causes duplicates or gaps.
- **Stop early** once you have enough data to answer. If `Total` is 5,000 and the user asked for "top 5 by spend", stop after page 1.

## Response-size limits

- CSV output is capped at **25 KB of characters** per call; truncation happens at row boundaries (no partial rows).
- A hard cap of **1,000 rows per page** is also applied regardless of `page_size`.
- If you see `⚠️ **TRUNCATED**`: narrow the query (shorter date range, tighter `filters`, smaller `page_size`) and retry. Do not silently present truncated data as complete.

## Typical flows

**"Top-spending content last week."**
```
get_top_campaign_content_report(
  account_id=...,
  start_date="YYYY-MM-DD",
  end_date="YYYY-MM-DD",
  sort_field="spent",
  sort_direction="DESC",
  page_size=20
)
```
Summarize the top 3–5 rows in prose, including absolute spend and share of total.

**"Why is CPC up on campaign X?"**
1. Call `get_campaign_history_report` with `account_id`, `start_date`, `end_date` for a window spanning before and after the change. History returns per-campaign time-series in one CSV; no sort, no filters — read campaign X's rows out of the combined result in post-processing.
2. Call `get_campaign_site_day_breakdown_report` with `account_id`, the same date range, and (if it works with the upstream API) `filters={"campaign_id": "<X>"}`. The MCP forwards `filters` verbatim to the Realize API, so unknown keys are silently ignored upstream — if filtering by `campaign_id` doesn't narrow the output, fall back to post-filtering the CSV.
3. Compare the recent period against the prior equivalent window. Report the delta and likely driver.

**"Break down my biggest campaign by site."**
1. Identify the biggest campaign (via `campaigns` skill or `get_top_campaign_content_report`).
2. Call `get_campaign_site_day_breakdown_report` with `account_id`, date range, `sort_field="spent"`, and `filters={"campaign_id": "<top>"}` if upstream accepts that key. Otherwise pull broadly and filter to the chosen `campaign_id` in post-processing.
3. Report top sites by spend, CTR, and CPC.

> **`filters` is a passthrough.** The MCP does not validate filter keys — it merges your `filters` dict directly into the upstream API's query string. If a key isn't recognized upstream, it's silently ignored (you won't get an error from the MCP). Always sanity-check whether a filtered call actually narrowed the result before trusting the scope.

## Interpretation guidelines

1. **Always translate relative dates.** "Last week" → explicit ISO dates in the call, and echo the date range back in your summary.
2. **Cite numbers, not adjectives.** "Top-performing" is meaningless without the spend/CTR figure next to it.
3. **Sanity-check totals.** If `Total: 0` came back, say so explicitly — don't make up narrative from an empty report.
4. **Flag missing sort support.** `get_campaign_history_report` accepts no sort/filter; if the user asked for "sorted history", explain that history returns API default order and offer to re-pull via the breakdown report instead.

## Gotchas

- **CSV, not JSON.** Report tools differ from campaign/account tools in response format.
- **Large reports** (`page_size=100` × many pages) are slow and risk truncation. Ask before paginating beyond the first 3 pages.
- **Silent filter passthrough.** `filters` dict keys are forwarded to the upstream API without MCP-side validation — an unrecognized key gets ignored silently, making the call look like it filtered when it didn't. Always verify `Total` matches the expected narrowed scope.
