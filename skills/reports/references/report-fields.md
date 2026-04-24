# Report Fields Reference

Quick lookup for what each Realize MCP report returns. Field names below are **illustrative, not schema-derived** — they should be cross-checked against real CSV output before any production use. Treat them as a starting hypothesis, not ground truth.

## `get_top_campaign_content_report`

Returns the top-performing content items across the account (aggregated across campaigns).

| Column | Meaning |
|---|---|
| `item_id` | Opaque item/creative identifier |
| `campaign_id` | Opaque campaign identifier the item belongs to |
| `thumbnail_url` | Creative thumbnail |
| `title` / `description` | Creative copy |
| `impressions` | Rendered impressions in the window |
| `clicks` | Clicks in the window |
| `spent` | Spend in account currency |
| `ctr` | Click-through rate (clicks / impressions) |
| `cpc` | Cost per click (spent / clicks) |
| `cvr` | Conversion rate (if conversions tracked) |

Sort: `clicks`, `spent`, `impressions`.

## `get_campaign_breakdown_report`

Aggregates metrics per campaign for the window.

| Column | Meaning |
|---|---|
| `campaign_id` | Opaque campaign identifier |
| `campaign_name` | Human-readable name |
| `status` | e.g., `RUNNING`, `PAUSED`, `TERMINATED` |
| `impressions`, `clicks`, `spent`, `ctr`, `cpc`, `cvr` | Performance metrics |
| `conversions`, `conversions_value` | If conversion tracking is configured |

Sort: `clicks`, `spent`, `impressions`. Filter: by campaign status or date range.

## `get_campaign_history_report`

Daily time-series for one or more campaigns. **No sort, no filter** — API default order only (usually ascending by date).

| Column | Meaning |
|---|---|
| `date` | ISO date of the row |
| `campaign_id` | Opaque campaign identifier |
| `impressions`, `clicks`, `spent`, `ctr`, `cpc`, `cvr` | Daily metrics |

Use this for trend analysis (e.g., "why did CPC spike last Tuesday").

## `get_campaign_site_day_breakdown_report`

Per-site, per-day breakdown. Good for finding which publishers/days drove campaign performance.

| Column | Meaning |
|---|---|
| `date` | ISO date |
| `site` | Publisher domain or placement identifier |
| `campaign_id` | Opaque campaign identifier |
| `impressions`, `clicks`, `spent`, `ctr`, `cpc`, `cvr` | Metrics for that (site, day, campaign) tuple |

Sort: `clicks`, `spent`, `impressions`. Filter: by date range, campaign, or site.

## Metadata header line

Every CSV starts with:

```
Records: <returned> | Total: <all matching> | Page: <n> | Size: <page_size>
```

Use `Total` to decide whether to paginate and to cite scope in the summary.
