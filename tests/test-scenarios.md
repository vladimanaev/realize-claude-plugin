# Test Scenarios

Manual QA checklist. Run each scenario against a real Realize test account and confirm the expected behavior. Scenarios are roughly ordered from simplest to most involved; later ones depend on state established by earlier ones (a resolved `account_id`, a known campaign, etc.).

---

## 1. Fresh install + OAuth

**Setup:** No prior Realize MCP configured. Fresh Claude Code session.

**User prompt:**
> "List my Realize accounts."

**Expected behavior:**
1. Claude detects the `realize-mcp` server in `.mcp.json`, attempts to connect.
2. Browser opens to Taboola SSO login.
3. After successful login, Claude calls `search_accounts` and returns a list of matching accounts with the opaque `account_id` string (e.g., `advertiser_12345_prod`) for each.

**Pass criteria:** OAuth browser flow completes without manual intervention; at least one account is returned.

---

## 2. Account selection with pagination

**User prompt:**
> "Find all accounts with 'test' in the name."

**Expected behavior:**
1. `search_accounts(query="test", page_size=10, page=1)`.
2. If results exceed 10, Claude offers to fetch additional pages (`page=2, 3, ...`) rather than bumping `page_size` beyond the hard cap of 10.
3. Claude lists account names + opaque `account_id` strings, asks which to use.

**Pass criteria:** Hard cap of `page_size=10` is respected; pagination is offered, not forced.

---

## 3. List campaigns for an account

**Prerequisite:** `account_id` resolved from scenario 1 or 2.

**User prompt:**
> "Show me my active campaigns."

**Expected behavior:**
1. Claude reuses the cached `account_id` (does not re-run `search_accounts`).
2. Calls `get_all_campaigns(account_id=...)`.
3. Filters to running/active campaigns by inspecting the `status` field (exact enum from the API response) and summarizes: count, combined spend, names of top few.

**Pass criteria:** No duplicate `search_accounts` call; summary is prose, not raw JSON dump.

---

## 4. Inspect a single campaign + its items

**Prerequisite:** Know one `campaign_id` from scenario 3.

**User prompt:**
> "Tell me about campaign <ID> and what creatives it's running."

**Expected behavior:**
1. `get_campaign(account_id=..., campaign_id=<ID>)`.
2. `get_campaign_items(account_id=..., campaign_id=<ID>)`.
3. Summary includes: objective, budget, status, creative count, any paused/rejected items flagged.

**Pass criteria:** Both tools are called; item-level status anomalies (if any) are surfaced.

---

## 5. Top-content report (sort by spend)

**User prompt:**
> "Which pieces of content drove the most spend in the last 7 days?"

**Expected behavior:**
1. Claude translates "last 7 days" to explicit ISO dates and echoes the range back.
2. Calls `get_top_campaign_content_report` with `sort_field="spent"`, `sort_direction="DESC"`, appropriate date params.
3. Parses the CSV, cites `Total` in the summary, lists the top 3–5 items with spend, clicks, CTR.

**Pass criteria:** Date window is explicit; output is prose with numbers (not CSV dumped verbatim); `Total` is cited.

---

## 6. Campaign breakdown report with filters

**User prompt:**
> "Break down spend by campaign for the last 30 days, only running campaigns."

**Expected behavior:**
1. `get_campaign_breakdown_report` with `account_id`, date range, and `filters={"campaign_status": "RUNNING"}` (or whatever key the upstream API uses for status — Claude may try a couple and verify `Total` changes).
2. Returns CSV, summarized as a ranked list with spend per campaign.

**Pass criteria:** `filters` param is used in the tool call (not post-hoc filtering alone) and `Total` reflects a narrowed scope vs. an unfiltered call. If upstream silently ignores the filter key, Claude falls back to post-processing and discloses the fallback to the user.

---

## 7. Campaign history report — no sort/filter

**User prompt:**
> "Give me campaign <ID>'s history for the last 2 weeks, sorted by spend descending."

**Expected behavior:**
1. Claude recognizes `get_campaign_history_report` does **not** accept sort/filter.
2. Calls the tool without sort, returns API default order (usually ascending by date).
3. Explains the limitation: "History comes back in date order — I can't sort it server-side. Would you like me to re-pull via the breakdown report, or reorder the rows locally in my summary?"

**Pass criteria:** Claude does not attempt an unsupported sort; it explains the limitation and offers an alternative.

---

## 8. Site/day breakdown with pagination

**User prompt:**
> "Show me the top 50 site/day rows for campaign <ID> this week."

**Expected behavior:**
1. `get_campaign_site_day_breakdown_report` with `page_size=50`, `sort_field="spent"`.
2. Summary cites `Total` and pages correctly if needed.
3. If further pages needed, `page_size` stays at 50 (not changed mid-flow).

**Pass criteria:** `page_size` is constant across pages; total scope is cited.

---

## 9. Write-intent request → create-campaign skill

**User prompt:**
> "Create a new prospecting campaign with a $500/day budget targeting the US."

**Expected behavior:**
1. The `create-campaign` skill activates.
2. Claude states up-front that no MCP tool currently exists for creating a campaign, and offers to walk the user through the Realize console instead.
3. Walks through the UI steps (Campaigns → New Campaign → objective → budget → targeting → creatives → Launch).
4. Offers to verify via `get_campaign` once the user says they've created it.

**Pass criteria:** **Claude does not fabricate a tool call for an action the MCP doesn't support.** No invented parameters, no made-up tool names. UI walkthrough is offered.

---

## 10. Error handling — invalid account_id

**User prompt:** (after manually corrupting the cached `account_id`, or just passing a bogus one)
> "Pull campaigns for account bogus_account_xyz."

**Expected behavior:**
1. MCP returns an error (invalid account_id).
2. Claude surfaces the error verbatim.
3. Offers to re-run `search_accounts` to resolve a valid `account_id`.

**Pass criteria:** Error is not swallowed or hallucinated around; recovery path is offered.

---

## 11. Error handling — empty report window

**User prompt:** Pick a date range that predates the account's earliest data (e.g., 2015).
> "Show me top content from 2015."

**Expected behavior:**
1. CSV returns `Records: 0 | Total: 0 | ...`.
2. Claude does **not** fabricate a narrative. Says explicitly: "No records found for 2015 — either no campaigns ran in that window or the account is newer than that."

**Pass criteria:** Empty-result honesty; no hallucinated data.
