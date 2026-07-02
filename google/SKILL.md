# Google Ads — Skill

Claude can run Google campaigns two ways — via browser automation (works today, no API approval needed) or via the Google Ads API (requires a developer token). Both support draft/paused mode.

---

## Method 1: Browser automation (no API required)

Claude drives [ads.google.com](https://ads.google.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [Google Ads](https://ads.google.com) in Chrome before starting
3. Your Customer ID — 10-digit number shown in the top right of Google Ads (format: XXX-XXX-XXXX)

### What Claude does

Claude navigates Google Ads, creates a campaign, sets campaign type, bidding, audience or keywords, budget, and ad copy. Before activating, it:
- Sets campaign status to **Paused** (won't serve, won't spend)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

Google Ads also has a formal **Campaign Drafts** feature (under a live campaign → Drafts) — Claude can use this for testing changes to existing campaigns.

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| Smart campaigns prompt on account creation | Dismisses — uses Expert Mode |
| "Optimize campaign" suggestions | Skips all auto-apply recommendations |
| Display Network checked by default on Search | Unchecks under Networks settings |
| Search Network partners on by default | Unchecks unless you want it |
| Broad match keywords by default | Wraps terms in `"quotes"` for phrase or `[brackets]` for exact unless you specify |
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern same as LinkedIn |

### Campaign setup prompt

```
Set up a Google Ads campaign in the UI — leave it Paused, don't publish.
Campaign type: Search / Display / YouTube / Performance Max
Goal: Awareness / Leads / Website traffic / Conversions
Keywords or audience: [keyword list or audience description]
Locations: [countries or cities]
Budget: $[amount] daily
Bidding: Maximize clicks / Target CPA $[amount] / Manual CPC
Ad copy: [headline 1, headline 2, headline 3 / descriptions — or ask Claude to generate]
Final URL: [your landing page]
```

### Delete after QA

```
Delete the paused Google Ads campaign we just created — campaign name: [name]
```

Claude navigates to the campaign, selects it, and removes it.

---

## Method 2: Google Ads API

Requires a developer token from a Google Ads Manager (MCC) account. Basic Access is granted automatically; Standard Access (for production scale) requires a separate review. As of 2026, Google has a backlog on developer token applications.

### Prerequisites

- Google Ads Manager account (MCC)
- Developer token from API Center in your MCC
- OAuth2 credentials (client ID + secret from Google Cloud Console)
- `GOOGLE_ADS_CUSTOMER_ID` — 10-digit account ID without dashes

Install the client library:

```bash
pip install google-ads
```

Set up `google-ads.yaml`:

```yaml
developer_token: YOUR_DEVELOPER_TOKEN
client_id: YOUR_CLIENT_ID
client_secret: YOUR_CLIENT_SECRET
refresh_token: YOUR_REFRESH_TOKEN
login_customer_id: YOUR_MCC_ID
```

### Campaign setup prompt

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

### Dry run mode

Google Ads API supports `validate_only: true` on mutate operations — Claude sends the full campaign structure for validation without creating anything.

```
Create a Google Search campaign for [audience] — validate only, don't submit.
```

Claude returns: full mutate request payload, API validation result (policy checks, character limits, targeting conflicts), estimated weekly impressions.

### Delete after QA

```
Delete the Google Ads campaign with resource name [campaigns/XXXXXXXXX]
```

Claude calls the CampaignService remove operation.

### Key settings Claude enforces

- `network_settings.target_search_network: true`, `target_content_network: false` for Search (no Display Network bleed)
- Ad rotation: `OPTIMIZE`
- Campaign status: `PAUSED` until you explicitly confirm launch
- Conversion tracking: Claude warns if no conversion action is configured before launching a conversion-goal campaign
