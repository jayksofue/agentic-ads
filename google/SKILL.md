# Google Ads — Skill

Claude connects to the Google Ads API to create search, display, and YouTube campaigns.

## Prerequisites

1. **Google Ads API access (requires application)**

   A developer token is required and must be applied for from a Google Ads Manager (MCC) account. Basic Access is granted automatically; Standard Access (unlimited operations) requires a separate review. As of 2026, Google has a backlog — expect delays.

   > **No developer token yet?** Claude can drive the Google Ads UI via browser automation while you wait for API access. Just say "use browser mode."

   Once you have access:
   - A Google Ads Manager account (MCC)
   - Developer token from API Center in your MCC
   - OAuth2 credentials (client ID + secret from Google Cloud Console)
   - `GOOGLE_ADS_CUSTOMER_ID` — your 10-digit account ID (no dashes)

2. **Install the Google Ads client library**:

   ```bash
   pip install google-ads
   ```

3. **Set up `google-ads.yaml`**:

   ```yaml
   developer_token: YOUR_DEVELOPER_TOKEN
   client_id: YOUR_CLIENT_ID
   client_secret: YOUR_CLIENT_SECRET
   refresh_token: YOUR_REFRESH_TOKEN
   login_customer_id: YOUR_MCC_ID
   ```

## Campaign setup prompt

```
Create a Google Ads campaign:
- Customer ID: [your account ID]
- Campaign type: SEARCH / DISPLAY / VIDEO / PERFORMANCE_MAX
- Goal: AWARENESS / LEADS / WEBSITE_TRAFFIC / CONVERSIONS
- Audience: [keywords, topics, custom intent, remarketing list]
- Locations: [countries, cities]
- Budget: $[amount] daily
- Bidding: TARGET_CPA $[amount] or MAXIMIZE_CLICKS or TARGET_ROAS
- Ad copy: [headlines, descriptions, final URL]
- Dry run: yes/no
```

## Dry run mode

Google Ads API supports `validate_only: true` on mutate operations — Claude sends the full campaign structure for validation without creating anything.

```
Create a Google Search campaign for [audience] — validate only, don't submit.
```

Claude returns:
- Full mutate request payload
- API validation result (policy checks, character limits, targeting conflicts)
- Estimated weekly impressions based on keyword planner data
- Explicit confirmation required before live submission

## Ad types by campaign goal

| Goal | Recommended type | What Claude builds |
|---|---|---|
| B2B awareness | Display / YouTube | Responsive display ad + targeting |
| Lead gen | Search | Responsive search ad + keyword list |
| Retargeting | Display / PMax | Audience list + creative assets |
| App installs | App campaign | App ID + creative assets |

## Key settings Claude enforces

- `network_settings.target_search_network: true`, `target_content_network: false` for Search (no Display Network bleed)
- Ad rotation: `OPTIMIZE` (Google's default, best for performance)
- Conversion tracking: Claude will warn if no conversion action is configured before launching a conversion-goal campaign
