---
name: optimize-campaign
description: Diagnose an underperforming Realize campaign and prescribe fixes grounded in Taboola's official optimization playbook. Uses MCP read tools to pull the data, then recommends adjustments — creative rotation, bid changes, targeting, site blocklists, budget reallocation — that the user applies in the Realize console since MCP currently does not expose write tools. Activates on "why is this campaign underperforming?", "how do I improve CPA?", "what should I pause?", "my CTR is dropping", etc.
allowed-tools: ["Read", "Bash", "AskUserQuestion"]
---

# Optimize Campaign

Data-driven diagnosis and recommendation loop for an existing Realize campaign. Pulls performance data via MCP, compares against Taboola's published optimization thresholds, and prescribes concrete next actions.

## When to use

Trigger on any of:
- "Why is my campaign underperforming?" / "CPA is too high" / "CTR is dropping"
- "What should I pause / cut / scale up?"
- "How do I improve campaign X?"
- "Which creatives / sites / regions are hurting me?"
- "Should I raise my bid?" / "Is my budget too low?"

If the user asks to *create* a new campaign, route to the `create-campaign` skill instead.

## Prerequisites

- `account_id` resolved via the `accounts` skill.
- A specific `campaign_id` in scope, or the user asking "across my whole account". For single-campaign questions, identify the campaign first via the `campaigns` skill if needed.
- Enough data to reason about. Taboola's thresholds:
  - **Minimum daily spend:** $50/day. Below this, any diagnosis is premature — tell the user so explicitly.
  - **Minimum clicks per item before judging:** 500–1000. Below this, recommending a pause or scale action is unreliable.

If the data isn't there yet, say so. Don't prescribe fixes off a small sample.

## Diagnostic workflow

### 1. Establish the baseline (MCP reads)

| Goal | Tool | Key params |
|---|---|---|
| How is the campaign trending? | `get_campaign_history_report` | `account_id`, `start_date`, `end_date` spanning ~14–30 days |
| What are the top items driving spend / clicks? | `get_top_campaign_content_report` | `sort_field="spent"` or `"clicks"`, `sort_direction="DESC"` |
| Which campaigns in the account are healthy vs. struggling? | `get_campaign_breakdown_report` | Optionally `filters={"campaign_status": "RUNNING"}` |
| Where (site × day) is spend going? | `get_campaign_site_day_breakdown_report` | `sort_field="spent"`, optionally `filters={"campaign_id": "<id>"}` |

Cite `Total` in your summary so the user knows the scope of what you queried.

### 2. Identify the failure mode

Classify against the metric combinations in the official playbook:

| Pattern | Likely cause | Prescription |
|---|---|---|
| **Low CTR + low CPA** | Creative is dull but landing page converts | Increase budget / reach; add more title/thumbnail variations to find a stronger hook |
| **High CTR + low conversion rate** | Creative is misleading *or* landing page is broken / off-message | Fix landing page, or replace misleading creatives |
| **Low CTR + high CPA** | Creative is weak and the clicks that do come aren't worth it | Pause or replace the creative; don't just raise the bid |
| **High CTR + high conversion + high CPA** | You're winning users but paying too much per auction | Consider lowering CPC, tightening targeting to better-converting segments, or blocking expensive low-return sites |
| **Healthy across the board but low volume** | You're throttled by budget or bid | Increase daily budget; raise CPC; add title/thumbnail variations; expand targeting |

### 3. Drill into the dimension

Taboola's article emphasizes that *user experiences and behaviors differ across devices, locations, and content types*. Before prescribing a campaign-level fix, check whether the problem lives in a specific slice:

- **Device** — does the campaign run across desktop and mobile? Is one starkly worse?
- **Location** — any geo consistently driving higher CPA than the others?
- **Site / publisher** — use `get_campaign_site_day_breakdown_report` to find sites consuming spend without converting.
- **Time of day** — day-parting patterns that warrant a separate campaign for different windows.
- **Creative (item)** — which items are below the 500–1000 click threshold with poor CVR?

### 4. Prescribe — and name the action

The MCP currently has no write tools, so every prescription is a **user action in the Realize console**. Be explicit about what they should change and where:

- **Pause a low-performing item** — Campaigns → open the campaign → Campaign Inventory → toggle the item's status.
- **Raise CPC / adjust bid** — Campaigns → open the campaign → Bidding.
- **Increase daily budget** — Campaigns → open the campaign → Budget.
- **Add creative variations** — Campaigns → Campaign Inventory → **+New Item** (recommended: 5–10 title/thumbnail variations per URL; 3–10 items per campaign for algorithm testing).
- **Block a site** — Campaigns → open the campaign → Advanced Options → Block Sites.
- **Tighten targeting** — Campaigns → open the campaign → Location / Platform / Audiences.
- **Isolate a top performer** — create a new campaign containing only the winning item(s) at an optimized CPC so you control its bid/budget independently.

After the user says they've made the change, offer to re-verify: pull `get_campaign` / `get_campaign_items` to confirm the new state, or re-run the relevant report after a data window (typically another 3–7 days).

## Prescription rules — quick reference

Condition-action rules from the official playbook:

- **≥500–1000 clicks on an item + low CVR vs. siblings** → pause the item.
- **≥500–1000 clicks on an item + high CVR vs. siblings** → pause the others, create a new campaign around the winner at an optimized CPC.
- **Good overall performance, low volume** → raise CPC, add 5–10 title/thumbnail variations, expand targeting.
- **CPA above goal** → tighten targeting, block low-return sites, pause low-performing items. Don't simply raise the bid.
- **Below $50/day in spend** → increase budget *before* drawing conclusions; you don't have enough data yet.
- **Campaign <7–10 days old** → avoid major optimizations; the algorithm is still learning. Surface this caveat when the data window is too recent.

## Metrics cheat sheet

| Metric | What it tells you |
|---|---|
| **CTR** | Whether the creative (title + thumbnail) is interesting enough to click. High CTR vs. siblings = strong hook. |
| **Conversion / Engagement rate** | Whether the clicks you bought actually deliver on the goal post-click. Landing-page quality and creative honesty both show up here. |
| **CPA** | Whether each acquisition costs what you can afford. Benchmark against your stated willingness-to-pay, not an industry average. |
| **Spend per site** | Which publishers are extracting your budget without returning value. Primary input for a blocklist decision. |

## Gotchas

- **Don't optimize against a too-small sample.** Below 500–1000 clicks per item or $50/day of spend, the numbers are noise. Say so rather than prescribing.
- **Don't just raise the bid to fix CPA.** The official playbook is explicit: raising CPC expands volume, but if conversion isn't working you'll spend more to acquire equally-bad users. Fix the creative or landing page first.
- **Don't hallucinate a pause tool.** The MCP has no write tools today — every pause / bid change / creative swap is a user action in the Realize console. Name the exact UI path and wait for the user to confirm before re-verifying.
- **Respect the learning phase.** Campaigns under 7–10 days old haven't established a baseline; major changes now will reset the algorithm's learning.
- **Changes need time to manifest.** After the user applies a change, wait 3–7 days of fresh data before judging the new state. Don't re-diagnose the next hour.
