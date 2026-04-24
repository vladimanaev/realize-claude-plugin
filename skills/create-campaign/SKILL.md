---
name: create-campaign
description: Guide the user through creating or editing a Realize campaign in the Realize console. Activates on any write-intent request (create, edit, pause, duplicate, delete) when the current MCP release does not expose a tool for that action. Grounded in Taboola's official setup guide — uses the exact field names, enum values, budget-vs-bid-strategy rules, and learning-phase recommendations. Falls back to a UI walkthrough rather than fabricating a tool call; offers MCP verification once the user is done.
allowed-tools: ["Read", "AskUserQuestion"]
---

# Create Campaign (UI Guidance)

This skill activates when the user asks for an action — campaign creation, editing, pausing, budget changes, creative swaps — that **no current MCP tool supports**. It walks the user through the equivalent steps in the **Realize console** using the setup flow as documented in Taboola's [official setup guide](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign).

After the user says they've completed the UI flow, this skill hands back to the `realize-analyst` agent to verify the new state via the MCP tools that *do* exist (e.g., `get_campaign`, `get_all_campaigns`).

When upstream adds new tools that cover an action this skill currently handles via UI, update the agent's Tool Reference and trim the affected walkthrough here in an explicit PR.

## Trigger phrases

Any of: *create, make, set up, launch, edit, update, change, pause, resume, duplicate, clone, delete, remove, increase the budget, swap the creative*.

## Response pattern

1. **Name the gap up front** (one sentence): *"There's no MCP tool today for that action, so I can't make the change directly — but I can walk you through it in the Realize console and then verify the result."*
2. **Ask the key parameters** via `AskUserQuestion` — only the ones you need for the specific action. For a new campaign, lean on the enums and rules below so the user isn't guessing. For a campaign-improvement request, route to `optimize-campaign` first if the user has an existing campaign to diagnose.
3. **Walk through the UI step by step**, using the exact labels from the console. Stick to the walkthroughs below; don't invent paths.
4. **Offer a verification step**: "Once you've saved it and it's through review (24–48 hours), tell me and I'll pull the campaign via MCP to confirm the settings match."

## Creating a new campaign — full walkthrough

Source: Taboola "Setting Up a New Campaign".

### Step 1 — Navigate

1. Log in to the Realize console.
2. Open **Campaigns** (left nav).
3. Click **+New** → **Campaign**. The **New Campaign** page opens.

### Step 2 — Required fields

Ask the user for these before you dictate the walkthrough. Every field below is required before the campaign can be submitted for review.

| Field | Accepted values | Notes |
|---|---|---|
| **Campaign Name** | Any text | Internal identifier. |
| **Branding Text** | Any text | "Site name or product you're promoting." Shown publicly under each item. |
| **Marketing Objective** | `Reach`, `Engagement`, `Leads`, `Online Purchases`, `App Promotion` | See enum below — pick the one that matches the user's goal. |
| **Scheduling** | Start date/time; optional end date; 24/7 or custom hours | Default: start immediately upon approval, 24/7. Recommended minimum runtime: **7–10 days** to establish a baseline. |
| **Bid Strategy** | `Maximize Conversions`, `Enhanced CPC`, `Fixed Bid` | Drives the budget minimums — see the Bid × Budget rules below. |
| **Budget** | Currency amount (daily or monthly) | Minimum depends on bid strategy — see below. |

### Step 3 — Marketing Objective (pick one)

Quote the user-facing description so the choice is easy:

- **Reach** — *"Increase awareness of your brand."*
- **Engagement** — *"Increase user engagement and page views."*
- **Leads** — *"Drive leads such as email sign-ups."*
- **Online Purchases** — *"Get people to buy your products."*
- **App Promotion** — *"Get people to install your app."*

### Step 4 — Bid Strategy × Budget rules

These are hard Taboola-published minimums. Don't let the user set a budget below them — they won't get enough data for the algorithm to stabilize, and the campaign will either underspend or churn through noise.

| Bid Strategy | Minimum daily budget | Also note |
|---|---|---|
| **Maximize Conversions** | **10× the CPA goal** per day | For learning-phase stability with conversion optimization. |
| **Enhanced CPC** | **5× the CPA goal** per day (and **150× CPA monthly**) | — |
| **Fixed Bid** | **5× the CPA goal** per day (and **150× CPA monthly**) | — |

**For non-conversion campaigns** (Reach / Engagement, where there's no CPA goal): target **100–200 clicks per day** as the minimum data volume. Budget = `CPC × desired clicks/day`. Example: $0.50 CPC × 100–200 clicks = $50–$100/day.

If the user names a conversion goal and a CPA target, compute the minimum daily budget and say the math out loud so they see where the number comes from.

### Step 5 — Targeting (recommend leaving broad for now)

Taboola's setup guide explicitly recommends *not* narrowing targeting on a fresh campaign — it limits reach and starves the algorithm of data.

| Field | What to recommend for a new campaign |
|---|---|
| **Location** | Leave blank or pick **Entire Country**. |
| **Platform** (desktop / mobile / tablet) | Leave blank — let data show what works. |
| **Connection Type**, **Operating System**, **Browser** | Leave blank. These are *optimization* levers for an existing campaign, not setup fields. |

### Step 6 — Advanced Options (keep clear at launch)

- **Block Sites** — leave empty initially. You need a site-by-site breakdown from real data (`get_campaign_site_day_breakdown_report`) before you know what to block.
- **Brand Safety Pre-Bid** — leave off unless the user has a specific regulated-vertical requirement.

Both settings can be applied later via the `optimize-campaign` flow.

### Step 7 — Audience Targeting (usually skip for new campaigns)

Types available: **My Audiences**, **Marketplace Audiences**, **Contextual**.

Taboola's guidance: **don't target specific audiences on a new campaign** — it collapses reach and distorts data. If the user insists on audience logic, use **audience suppression** (under Conversion settings) to *exclude* already-converted users, rather than *including* a narrow segment.

### Step 8 — Campaign Inventory (the ads themselves)

Required per ad:
- **Thumbnail image**
- **Headline**
- **Website URL**

Optional:
- **CTA Button**
- **Ad Description**

**Recommended volume: 3–10 ads per campaign** so the algorithm has variations to test. Creative best practices (from Taboola):

- Titles and thumbnails should **pre-qualify the click** — match what the user will find on the landing page. Misleading creatives spike CTR and tank conversion.
- Avoid generic CTAs like *"Click Here"* or *"While Supplies Last"* — use original, specific copy.
- For performance marketers: one URL + 5–10 title/thumbnail variations per campaign.
- For publishers/brands: up to 10–20 URLs, each with 5–10 variations.
- **Dynamic Keyword Insertion** is available for location, day-of-week, or device (e.g., *"People in {{location}} Can't Get Enough of This Razor"*).

### Step 9 — Submit and wait

1. Click **Continue** to submit for review.
2. Realize reviews the campaign in **24–48 hours**. The same window applies to any edit after launch.
3. Once approved, the campaign starts on its scheduled start time.

### Step 10 — Post-launch verification (MCP-backed)

Offer the user a verification step once the campaign is live:

- Pull `get_campaign(account_id=..., campaign_id=...)` to confirm the fields match what they entered.
- Pull `get_campaign_items(account_id=..., campaign_id=...)` to confirm the ads are attached and live.
- After **7–10 days** of data, hand off to `optimize-campaign` for the first performance review. Do not recommend optimizations before that — the algorithm is still in its learning phase.

## Editing an existing campaign

1. Open **Campaigns**, find the row, click the campaign name.
2. In the campaign detail view, click **Edit** on the section you want to change (Budget, Targeting, Campaign Inventory, Bid…).
3. Apply the change and **Save**. Edits trigger another **24–48 hour review**.

Then verify via MCP (`get_campaign` after review completes).

## Pause / resume

1. In the Campaigns list, toggle the status switch on the row, **or**
2. Open the campaign and use the **Pause** / **Resume** button.

Verify via `get_campaign` — the `status` field should reflect the change.

## Duplicate

1. In the Campaigns list, open the row's overflow menu (⋯).
2. Click **Duplicate**.
3. Edit the copy's name, budget, and targeting as needed, then **Continue** to submit.

## Verification hand-off

Once the user confirms completion, call the `realize-analyst` agent (or route back to `campaigns` / `reports` skills) to:

- Fetch the new/edited campaign via `get_campaign(account_id=..., campaign_id=...)` and read back the settings. Both params are required — never call `get_campaign` with only `campaign_id`.
- For pauses/resumes, confirm the `status` field in `get_campaign(account_id=..., campaign_id=...)` or list via `get_all_campaigns(account_id=...)`.
- For budget edits, echo the new `budget` / `daily_budget` field values.

## Gotchas

- **Never pretend a write happened.** If the user says "go ahead and create it", restate the limitation and offer the walkthrough instead. Fabricating a successful response is a trust-breaker.
- **Don't bypass the minimum budget rules.** A $10/day campaign with a $20 CPA goal will waste the $10 — Taboola published those minimums because below them the algorithm can't stabilize.
- **Don't suggest narrowing targeting at launch** to "focus on the right users" — it's the opposite of Taboola's guidance. Narrow *after* you have real data showing which segments underperform.
- **Don't invent Realize UI paths that aren't in this file.** For console steps not covered here (advanced audience builder, A/B tests, custom brand-safety rules), direct the user to Realize documentation rather than guessing.
- **Review cycle applies to edits, not just creation.** An edit to an existing live campaign still takes 24–48 hours to re-approve — set that expectation.
- **Data may lag briefly** in MCP results after UI changes. If `get_campaign` returns the old state right after a save, wait a minute and retry once.
