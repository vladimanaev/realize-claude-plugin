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
> "Create a new Online Purchases campaign with a $25 CPA target."

**Expected behavior:**
1. The `create-campaign` skill activates.
2. Claude states up-front that no MCP tool currently exists for creating a campaign, and offers to walk the user through the Realize console instead.
3. Asks for the required fields from Taboola's setup guide: **Campaign Name**, **Branding Text**, **Marketing Objective** (offers the 5-value enum: Reach / Engagement / Leads / Online Purchases / App Promotion), **Bid Strategy** (Maximize Conversions / Enhanced CPC / Fixed Bid), and Budget.
4. Applies the bid-strategy budget rule: for **Maximize Conversions** with a $25 CPA target, recommends **≥$250/day** (10× CPA). For Enhanced CPC / Fixed Bid, recommends **≥$125/day** (5× CPA) with a $3,750 monthly minimum (150× CPA).
5. Walks through the UI navigation path `Campaigns → +New → Campaign`, covering the required and optional sections, and recommends **leaving targeting / audiences / blocklists broad** at launch (per official guide).
6. Calls out the **24–48 hour review** and the **7–10 day learning phase** before offering a follow-up optimization.
7. Offers to verify via `get_campaign(account_id=..., campaign_id=...)` once the campaign clears review.

**Pass criteria:** **Claude does not fabricate a tool call for an action the MCP doesn't support.** Budget minimums are enforced against the CPA target (not guessed). Targeting guidance matches the official "stay broad at launch" recommendation. The 24–48 hr review and 7–10 day learning phase are both surfaced.

---

## 9a. Optimization request — adequate data

**Prerequisite:** A campaign with at least two items, one with ≥500 clicks and clearly worse CVR than its siblings.

**User prompt:**
> "Campaign 12345 is underperforming. What should I do?"

**Expected behavior:**
1. The `optimize-campaign` skill activates.
2. Claude resolves `account_id`, then pulls `get_campaign_breakdown_report` and `get_campaign_site_day_breakdown_report` for the campaign, plus `get_top_campaign_content_report` at the account level for context. Date window is echoed in the summary.
3. Checks thresholds before prescribing: confirms daily spend ≥$50 and at least one item has ≥500–1000 clicks. If either is missing, says so explicitly and does not prescribe.
4. Classifies the failure mode against the prescription rules (CTR × CVR × CPA) and names it (e.g., "High CTR, low conversion rate — this is typically a landing-page or creative-honesty issue, not a bid issue").
5. Prescribes a concrete UI action with the exact console path (e.g., "Pause item 887003: Campaigns → open 12345 → Campaign Inventory → toggle item status").
6. Offers to re-verify via MCP after the user applies the change AND 3–7 days of fresh data have accrued.

**Pass criteria:** Classification cites at least two of CTR / CVR / CPA with real numbers. Prescription includes a specific console path. Claude does **not** recommend simply "raise the bid" as the first response to high CPA.

---

## 9b. Optimization request — insufficient data (learning phase)

**Prerequisite:** A campaign <7 days old, or one with <500 total clicks.

**User prompt:**
> "Why is my brand-new campaign not getting conversions? Should I pause it?"

**Expected behavior:**
1. The `optimize-campaign` skill activates.
2. Claude pulls the campaign's history via MCP, sees the data is thin (either the age window or the click total is under threshold), and **refuses to prescribe**.
3. Surfaces the specific threshold that was missed: "the algorithm's learning phase is 7–10 days" or "Taboola recommends at least 500–1000 clicks per item before judging performance".
4. If spend is below $50/day, recommends **increasing the daily budget** before drawing any further conclusions.
5. Offers to revisit the diagnosis once the threshold is met.

**Pass criteria:** Claude does **not** recommend a pause, bid change, or targeting change on insufficient data. Names the exact threshold(s) that haven't been met.

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
