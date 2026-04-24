# Realize MCP — Backlog

Capabilities missing from the current [Realize MCP](https://github.com/taboola/realize-mcp) (`streamable-http`, read-only, 11 tools) that would let this plugin automate workflows users currently have to execute manually at [ads.realizeperformance.com](https://ads.realizeperformance.com). Sourced from Taboola's official best-practice documentation.

Full rationale and per-item detail: [`docs/realize-best-practices-gap.md`](docs/realize-best-practices-gap.md).

## Campaign lifecycle

- `create_campaign` — build a campaign with name, branding, objective, scheduling, bid strategy, budget. [Setting Up a New Campaign](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign)
- `update_campaign` — edit any field post-launch. [Setting Up a New Campaign](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign)
- `pause_campaign` / `resume_campaign` — toggle status. [How to Improve Campaign Performance](https://www.taboola.com/help/en/articles/3878108-how-to-improve-campaign-performance)
- `duplicate_campaign` — clone with optional overrides.
- `submit_campaign_for_review` — programmatic submission replacing the `Continue` button.
- `get_review_state` — pending / Fully launched / Partially launched / Rejected, per-item. [Campaign Review Process](https://www.taboola.com/help/en/articles/3878200-campaign-review-process)
- `get_rejection_reasons` — policy-violation codes on rejected campaigns/items.

## Campaign items (creatives)

- `add_campaign_item` — create ad (thumbnail + headline + URL; optional CTA / description).
- `update_campaign_item` — edit post-launch.
- `pause_campaign_item` / `resume_campaign_item` — the single most-common optimization write.
- `upload_asset` — image/video upload to asset library.
- `generate_creative_variations` — invoke GenAI Ad Maker. [A/B Testing Campaign Items](https://help.taboola.com/hc/en-us/articles/360063535593-A-B-Testing-Campaign-Items)
- `set_traffic_allocation_mode` — `EVEN` for formal A/B test vs. algorithm-biased default. [A/B Testing Campaign Items](https://help.taboola.com/hc/en-us/articles/360063535593-A-B-Testing-Campaign-Items)

## Targeting & audiences

- `get_targeting_state` — read current Location / Platform / Device / Connection / OS / Browser on a campaign (currently unverified whether `get_campaign` returns this).
- `update_targeting` — adjust any targeting dimension.
- `list_audiences` / `create_audience` — manage account audiences. [Retargeting Users Who Clicked on Campaign Ads](https://www.taboola.com/help/en/articles/3878045-retargeting-users-who-have-clicked-on-campaign-ads)
- `attach_audience_to_campaign` — add `My Audiences` / Marketplace / Contextual / suppression audiences. [Setting Up a New Campaign](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign)
- `setup_campaign_clicker_retargeting` — the IMG-tag-in-3rd-party-pixels flow with ~2-week audience build. [Retargeting Users Who Clicked on Campaign Ads](https://www.taboola.com/help/en/articles/3878045-retargeting-users-who-have-clicked-on-campaign-ads)
- `get_blocklist` / `update_blocklist` — read/add/remove sites from Block Sites. [How to Improve Campaign Performance](https://www.taboola.com/help/en/articles/3878108-how-to-improve-campaign-performance)
- `toggle_brand_safety_pre_bid` — enable/disable the brand-safety filter. [Setting Up a New Campaign](https://www.taboola.com/help/en/articles/10473049-setting-up-a-new-campaign)

## Bid & budget

- `update_bid` — adjust campaign CPC.
- `update_daily_budget` / `update_monthly_budget` — change budget caps.
- `update_traffic_source_bid` — per-source CPC overrides. [How to Improve Campaign Performance](https://www.taboola.com/help/en/articles/3878108-how-to-improve-campaign-performance)

## Conversion tracking (entire category is missing)

- `get_pixel_status` — read indicator (`Active` / `No Recent Activity` / `No Activity` / `Waiting for Pixel` / `Not Installed`) + last-event timestamp. [Taboola Pixel Overview](https://www.taboola.com/help/en/articles/6248618-taboola-pixel-overview)
- `list_conversions` — enumerate URL-based / event-based / codeless conversions on the account. [Defining and Creating Conversions](https://developers.taboola.com/pixel/docs/defining-conversions)
- `create_conversion` / `update_conversion` — define new or edit existing.
- `update_conversion_attribution_windows` — click-through (1–30 days, default 30); view-through (1–24 hours, default 24). [Defining and Creating Conversions](https://developers.taboola.com/pixel/docs/defining-conversions)
- `toggle_total_conversions_flag` — which conversions feed SmartBid / Maximize Conversions (retroactively affects 30 days of reports). [Defining and Creating Conversions](https://developers.taboola.com/pixel/docs/defining-conversions)
- `get_conversion_events_report` — breakdown by event type (Purchase / Lead / AddToCart / etc.) — distinct from existing campaign reports.
- `get_tracking_code` / `apply_tracking_code` — manage the 16+ URL-parameter macros (`{campaign_id}`, `{site}`, `{platform}`, `{cachebuster}`, `${GDPR}`, etc.) at campaign- or per-ad-level with the "don't apply at both levels" rule. [URL Parameters for Tracking](https://developers.taboola.com/pixel/docs/url-params-for-tracking)

## Analytics (read-tool expansions)

- Period-over-period comparison in a single call (current reports are single-window; Claude fetches twice and diffs manually).
- Attribution-window / cohort analytics.
- Audience / segment performance breakdown.
- Scheduling / dayparting state inspection on a campaign.

## Notes

- Landing-page quality signals are called out in [Landing Page Best Practices for Performance Advertisers](https://www.taboola.com/help/en/articles/3878047-landing-page-best-practices-for-performance-advertisers) but are a human content-quality call — likely out of scope for MCP automation even long-term.
- When any of these lands upstream, follow the process in Part 4 of the gap doc: record it in the agent's Tool Reference, wire it into the right skill (or create a new one), add a test scenario, and strike the item from this backlog.
