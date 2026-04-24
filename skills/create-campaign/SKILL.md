---
name: create-campaign
description: Guide the user through creating or editing a Realize campaign in the Realize UI. Activates on any write-intent request (create, edit, pause, duplicate, delete) when the current MCP release does not expose a tool for that action. Falls back to a UI walkthrough rather than fabricating a tool call; offers MCP verification once the user is done.
allowed-tools: ["Read", "AskUserQuestion"]
---

# Create Campaign (UI Guidance)

This skill activates when the user asks for an action — campaign creation, editing, pausing, budget changes, creative swaps — that **no current MCP tool supports**. It walks the user through the equivalent steps in the **Realize console** (the web UI).

After the user says they've completed the UI flow, this skill hands back to the `realize-analyst` agent to verify the new state via the MCP tools that *do* exist (e.g., `get_campaign`, `get_all_campaigns`).

When upstream adds new tools that cover an action this skill currently handles via UI, update the agent's Tool Reference and trim the affected walkthrough here in an explicit PR.

## Trigger phrases

Any of: *create, make, set up, launch, edit, update, change, pause, resume, duplicate, clone, delete, remove, increase the budget, swap the creative*.

## Response pattern

1. **Name the gap up front** (one sentence): *"There's no MCP tool today for that action, so I can't make the change directly — but I can walk you through it in the Realize console and then verify the result."*
2. **Ask the key parameters** via `AskUserQuestion` (only what's needed for the specific action):
   - **New campaign**: objective, daily budget, bid strategy, targeting (countries, devices), creative source.
   - **Edit**: which campaign (`campaign_id`), which field.
   - **Pause/resume**: which campaign.
3. **Provide the step-by-step UI walkthrough** in numbered steps referencing actual Realize console navigation. (Keep steps short — 1 action per line.)
4. **Offer a verification step**: "Once you've saved it, tell me and I'll pull the campaign via MCP to confirm the settings match."

## UI walkthroughs

### Create a new campaign
1. Log in to the Realize console.
2. Navigate to **Campaigns → New Campaign**.
3. Pick the marketing objective (Awareness / Traffic / Conversions).
4. Set daily budget and bid strategy.
5. Configure targeting (country, device, day-parting if needed).
6. Add creatives (upload or select from library).
7. Review and **Launch**.

### Edit an existing campaign
1. Open **Campaigns**, find the row, click the name.
2. In the campaign detail view, click **Edit** on the section you want to change (Budget, Targeting, Creatives…).
3. Apply the change and **Save**.

### Pause / resume
1. In the Campaigns list, toggle the status switch on the row, **or**
2. Open the campaign and use the **Pause** / **Resume** button in the top-right.

### Duplicate
1. In the Campaigns list, open the row's overflow menu (⋯).
2. Click **Duplicate**.
3. Edit the copy's name, budget, and targeting as needed, then **Launch**.

## Verification hand-off

Once the user confirms completion, call the `realize-analyst` agent (or route back to `campaigns` / `reports` skills) to:

- Fetch the new/edited campaign via `get_campaign(account_id=..., campaign_id=...)` and read back the settings. Both params are required — never call `get_campaign` with only `campaign_id`.
- For pauses/resumes, confirm the `status` field in `get_campaign(account_id=..., campaign_id=...)` or list via `get_all_campaigns(account_id=...)`.
- For budget edits, echo the new `budget` / `daily_budget` field values.

## Gotchas

- **Never pretend a write happened.** If the user says "go ahead and create it", restate the limitation and offer the walkthrough instead. Fabricating a successful response is a trust-breaker.
- **Don't invent Realize UI paths that aren't in this file.** If the user's task requires console steps not listed here (e.g., advanced audience rules, A/B tests), say so and direct them to Realize documentation rather than guessing.
- **New items may take a few minutes to appear** in MCP read results after UI changes — if `get_campaign` returns the old state right after a save, wait a minute and retry once.
