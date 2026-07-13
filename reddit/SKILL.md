---
name: agentic-ads-reddit
description: Deploy Reddit ad campaigns via browser automation (ads.reddit.com) or Reddit Ads API v3 (allow-list required). Strong for subreddit/community targeting. NOT YET LIVE-VERIFIED — API field names below need sandbox confirmation before shipping (see safety banner). Use when working with Reddit ads specifically; loaded by the parent agentic-ads skill.
---

# Reddit Ads — Skill

Claude can run Reddit campaigns two ways — via browser automation (works today, no API approval needed) or via the Reddit Ads API v3 (requires OAuth + allow-list approval). Both support a paused/review state so you can check everything before spending.

Reddit is a strong fit for community-intent targeting: you point ads at specific subreddits (r/CryptoCurrency, r/fintech, r/ethfinance, r/stablecoins) rather than guessing at interests. Subreddit targeting is the platform's core advantage.

> **QA status (2026-07-09): 📝 Documented, not yet verified end-to-end. ⚠️ Do NOT ship live without a sandbox run first.**
> This has not been run through a live create→delete cycle from this environment. No Reddit Ads credentials are configured, and the Ads API additionally requires allow-list approval from Reddit's ads team.
>
> **⚠️ Safety-critical caveat on API field names.** The paused-state field name below is written as `is_enabled: false` (from one community integration reference), but a subsequent audit found other Reddit Ads API references using **`configured_status: "PAUSED" / "ACTIVE"`** instead. **Reddit's v3 reference is not publicly available for verification** (docs are gated behind allow-list approval), so I could not confirm which is correct from primary sources. **Before your first live create call, POST a paused campaign in Reddit's sandbox and verify the actual field name and values that Reddit accepts** — if the wrong key is used, Reddit may ignore it and default to ACTIVE (i.e., spend on save). Use the browser path (Method 1) until API field names are confirmed on your account.
>
> Budget units (micros), OAuth scopes, base URL, and endpoint paths below are cross-checked against multiple community integrations and Reddit's public help pages, but the same caveat applies — verify against a sandbox call before production.

---

## Method 1: Browser automation (no API required)

Claude drives [ads.reddit.com](https://ads.reddit.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [Reddit Ads Manager](https://ads.reddit.com) in Chrome before starting
3. A Reddit Ads account (created free at ads.reddit.com; a payment method must be on file before a campaign can leave draft)

### What Claude does

Claude navigates Ads Manager, creates the campaign, sets objective, community/interest targeting, budget, and attaches a post (a link/text/image post promoted as an ad). Before activating, it:
- Leaves the campaign **Paused** (draft equivalent — won't serve)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

### Structure

```
Account
└── Campaign      ← objective, budget, schedule
    └── Ad Group  ← targeting (communities/interests), bid, placement
        └── Ad    ← the promoted post (creative + headline + CTA + destination)
```

Targeting lives at the **ad group** level, not the campaign — same as Meta's ad set.

### Objectives

Reddit's objectives (pick based on the goal): **Brand Awareness**, **Reach**, **Traffic**, **Engagement**, **Video Views**, **Conversions**, **App Installs**, **Lead Generation**. (Catalog Sales is a Meta objective, not a Reddit one — do not confuse them.) For a cold audience with a landing page, use **Traffic**; for pure awareness with no click goal, use **Brand Awareness** / **Reach**.

### Targeting (Reddit-specific)

| Type | What it does | Notes |
|---|---|---|
| **Communities** | Show ads in specific subreddits | The platform's strongest lever — target r/CryptoCurrency, r/fintech, etc. directly |
| **Interests** | Reddit's interest categories | Broader than communities; pair with communities to narrow |
| **Keywords** | Match posts/searches by keyword | Contextual, not user-profile based |
| **Geography** | Countries, regions, metros, named locations | US supported |
| **Devices / OS / carriers** | Platform-level filters | |
| **Custom audiences** | Customer lists, retargeting from the Reddit Pixel, lookalikes | Requires the pixel installed first |

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern: `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set` |
| Community targeting search is async | Type the subreddit, wait for the suggestion list, click the match — don't assume the ID resolves instantly |
| "Advantage"-style auto-expansion | Reddit may pre-enable audience expansion / broadened targeting — turn it off for a controlled audience |
| Placement defaults | Feed + conversation placements on by default; conversation-only or feed-only is set under placements |
| Promoted post vs existing post | Can promote an existing post (paste its URL) or create a new promoted-only post |
| Payment method required to leave draft | A paused campaign can be built without spend, but publishing needs a billing method on file |
| Delete draft campaign | Campaign row → overflow menu → Delete (or Archive) → confirm |

### Campaign setup prompt

```
Set up a Reddit campaign in Ads Manager — leave it paused, don't publish.
Objective: Traffic / Brand Awareness / Conversions / Video Views / Engagement
Communities: [r/CryptoCurrency, r/fintech, ...]  (and/or interests, keywords)
Audience: [geo, devices]
Budget: $[amount] daily or lifetime
Post: [post URL to promote, or "write a new promoted post: [copy]"]
Destination URL: [landing page]
Audience expansion: off
```

### Delete after QA

```
Delete the paused Reddit campaign we just created — campaign name: [name]
```

Claude opens the campaign row overflow menu and deletes (or archives) it.

---

## Method 2: Reddit Ads API v3

Programmatic path — faster for bulk setup, no UI. Requires OAuth **and** allow-list approval.

- **Base URL:** `https://ads-api.reddit.com/api/v3/`
- **Docs:** `https://ads-api.reddit.com/docs/v3/`
- **Auth:** OAuth 2.0 Bearer token on every request (`Authorization: Bearer <token>`)
- **Rate limit:** ~1 request/second

### Prerequisites

1. **Register an app** at [reddit.com/prefs/apps](https://www.reddit.com/prefs/apps) → get `client_id` + `client_secret`.
2. **Request the Ads OAuth scopes** at authorization time: `adsread`, `adsedit`, `adsconversions`, `history`, `read`.
3. **Allow-list approval** — full campaign management (create/edit) requires Reddit's ads or partner team to approve your app. Read-only access is available sooner; write access is gated.
4. **OAuth flow:** authorize at `https://www.reddit.com/api/v1/authorize`, exchange the code at `https://www.reddit.com/api/v1/access_token`, refresh with the `refresh_token` grant when the access token expires.

Set credentials in your environment:

```
REDDIT_ADS_CLIENT_ID=...
REDDIT_ADS_CLIENT_SECRET=...
REDDIT_ADS_ACCESS_TOKEN=...
REDDIT_ADS_REFRESH_TOKEN=...       # exchange for a new access token via the refresh_token grant
REDDIT_ADS_ACCOUNT_ID=...
```

Reddit access tokens expire in ~1 hour — the `refresh_token` grant against `https://www.reddit.com/api/v1/access_token` (Basic auth = client_id:client_secret) gives you a new one. Automate the refresh in your runner or the second-hour of any long job will 401.

### What Claude does via API

Create in order: **campaign → ad group → ad**. Targeting and bid live on the ad group.

> ⚠️ **Every `is_enabled` below is the field name I used from one community integration. The audit flagged that Reddit Ads API v3 may actually use `configured_status: "PAUSED" / "ACTIVE"` instead** — Reddit's v3 spec is not publicly available for a cross-check. Test whichever name Reddit accepts on your account against a paused sandbox campaign before your first production call. If Reddit ignores an unknown key, your "paused" object may go live on save.

**Campaign** (start paused):
```json
{
  "objective": "TRAFFIC",
  "is_enabled": false,        // OR — CONFIRM AGAINST SANDBOX — "configured_status": "PAUSED"
  "budget_type": "DAILY",
  "budget_total_amount_micros": 10000000,
  "start_time": "2026-07-10T00:00:00Z"
}
```

**Ad group** (targeting + bid):
```json
{
  "campaign_id": "<CAMPAIGN_ID>",
  "is_enabled": false,        // OR — CONFIRM AGAINST SANDBOX — "configured_status": "PAUSED"
  "bid_strategy": "LOWEST_COST",
  "goal_type": "CLICKS",
  "targeting": {
    "geolocations": ["US"],
    "community_names": ["CryptoCurrency", "fintech"]
  }
}
```

**Key API behaviors (from community integrations — confirm on a live sandbox run):**
- **Budget and bids are in micros: 1 USD = 1,000,000 micros.** So `$10 = 10000000`. Not cents.
- Paused/active field: `is_enabled` OR `configured_status` — see safety caveat above. Whichever the account accepts, verify a paused create actually stays paused by GET-ing the object back before any activation call.
- `budget_type`: `DAILY` or `LIFETIME`.
- `bid_strategy`: `LOWEST_COST`, `COST_CAP`, or `MANUAL`. `COST_CAP` / `MANUAL` require `bid_amount_micros`.
- `goal_type`: `CLICKS`, `IMPRESSIONS`, `VIDEO_VIEWS`, etc. — must be compatible with the campaign objective.
- Targeting IDs (communities, interests, geos, devices) come from the API's targeting-search endpoints; resolve names to IDs before creating the ad group.

### Dry run mode

The Reddit Ads API has **no `validate_only` flag**. Two ways to preview without spending:
1. **Create the campaign and ad group in the paused state** (real objects, real IDs, but not serving) — then verify the readback shows the paused state you sent, and delete after QA (same idea as Meta).
2. **Preview an Ad** endpoint (`docs/v3/preview-an-ad`) — renders how the ad will look without publishing.

```
Create a Reddit campaign for [audience] — build it paused, GET the campaign back to confirm the paused-state field took effect, show me the payload, then wait for my explicit go.
```

Claude returns the full request bodies (campaign + ad group + ad), the resolved targeting IDs, the GET readback confirming paused state, and an explicit confirm prompt before any activation call.

### Delete after QA

Delete the ad group first, then the campaign (both are account-scoped in v3):
```
DELETE /api/v3/ad_accounts/{account_id}/ad_groups/{ad_group_id}
DELETE /api/v3/ad_accounts/{account_id}/campaigns/{campaign_id}
```
If DELETE is not permitted for your access level, disable then archive via whatever paused-state field your account accepts.

### Conversion tracking (Reddit Pixel + CAPI)

For a Conversions objective, install the **Reddit Pixel** on the destination and/or send events server-side via the **Conversions API** (`docs/v3/capi-direct-integration`). Same idea as Meta's pixel + CAPI: browser pixel plus server events recover conversions lost to blockers. Not needed for Traffic/Awareness QA.

### Key settings Claude enforces

- Campaign starts paused (verify readback) until you explicitly confirm launch
- Budget/bids always expressed in micros ($1 = 1,000,000)
- Audience expansion / broadened targeting off unless you ask for it
- Regulated categories: flag if the ad is finance/crypto so review expectations are set (Reddit reviews crypto ads and restricts some geos)
