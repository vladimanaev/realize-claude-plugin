# CSV Examples

Illustrative (fictional) sample outputs from the four Realize MCP report tools. Column sets are a starting hypothesis, not schema-derived — verify against real output before relying on specific field names. The surrounding format (titled header + `📊` metadata line + CSV) is taken from the upstream MCP source.

## `get_top_campaign_content_report`

```
🏆 **Top Campaign Content Report CSV** - Account: advertiser_12345_prod | Period: 2026-04-17 to 2026-04-23

📊 Records: 3 | Total: 47 | Page: 1 | Size: 3

item_id,campaign_id,title,impressions,clicks,spent,ctr,cpc
887001,12345,"10 tips for better sleep",540320,8104,812.40,0.015,0.100
887002,12345,"Morning routine of CEOs",498211,6203,620.30,0.0125,0.100
887003,67890,"Best noise-cancelling headphones 2026",410002,9884,1580.00,0.0241,0.160
```

Interpretation pattern:
> "Across 47 top items this week, the leader by spend is *Best noise-cancelling headphones 2026* in campaign 67890 ($1,580 / 9,884 clicks / $0.16 CPC). The two runners-up are both sleep-content items in campaign 12345 with identical $0.10 CPCs."

## `get_campaign_breakdown_report`

```
🏆 **Campaign Breakdown Report CSV** - Account: advertiser_12345_prod | Period: 2026-04-01 to 2026-04-23

📊 Records: 2 | Total: 2 | Page: 1 | Size: 20

campaign_id,campaign_name,status,impressions,clicks,spent,ctr,cpc
12345,"Sleep Products - Q2 Prospecting",RUNNING,1842111,23117,2311.70,0.01255,0.100
67890,"Headphones - Retargeting",RUNNING,820045,19770,3164.20,0.0241,0.160
```

## `get_campaign_history_report`

```
🏆 **Campaign History Report CSV** - Account: advertiser_12345_prod | Period: 2026-04-17 to 2026-04-23

📊 Records: 7 | Total: 7 | Page: 1 | Size: 20

date,campaign_id,impressions,clicks,spent,ctr,cpc
2026-04-17,12345,260110,3201,320.10,0.0123,0.100
2026-04-18,12345,265500,3310,331.00,0.0125,0.100
2026-04-19,12345,270122,3420,342.00,0.0127,0.100
2026-04-20,12345,255400,3190,319.00,0.0125,0.100
2026-04-21,12345,280011,3510,351.00,0.0125,0.100
2026-04-22,12345,260400,3250,325.00,0.0125,0.100
2026-04-23,12345,250900,3136,313.60,0.0125,0.100
```

Interpretation pattern:
> "Campaign 12345 held a stable 0.0125 CTR across Apr 17–23, with daily spend ranging $313–$351. No trend or anomaly — steady-state performance."

## `get_campaign_site_day_breakdown_report`

```
🏆 **Campaign Site Day Breakdown Report CSV** - Account: advertiser_12345_prod | Period: 2026-04-21 to 2026-04-21

📊 Records: 4 | Total: 312 | Page: 1 | Size: 4

date,site,campaign_id,impressions,clicks,spent,ctr,cpc
2026-04-21,nytimes.com,12345,95012,1420,142.00,0.01494,0.100
2026-04-21,cnn.com,12345,62014,780,78.00,0.01258,0.100
2026-04-21,wsj.com,12345,44211,655,65.50,0.01481,0.100
2026-04-21,foxnews.com,12345,36801,540,54.00,0.01467,0.100
```

Interpretation pattern:
> "On Apr 21 for campaign 12345, *nytimes.com* drove the most volume ($142 spend, 1,420 clicks, 1.49% CTR). The CTR spread across the top 4 sites was tight (1.26–1.49%), suggesting site mix is not the CPC variance driver here."

## Empty-result handling

```
🏆 **Campaign History Report CSV** - Account: advertiser_12345_prod | Period: 2026-04-17 to 2026-04-23

📊 Records: 0 | Total: 0 | Page: 1 | Size: 20

date,campaign_id,impressions,clicks,spent,ctr,cpc
```

Never fabricate narrative from an empty report. Say so explicitly:
> "No records returned for campaign 12345 between Apr 17 and Apr 23 — either the campaign wasn't running in that window or no data has been ingested yet."
