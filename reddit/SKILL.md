---
name: agentic-ads-reddit
description: Deploy Reddit ad campaigns via browser automation (ads.reddit.com) or Reddit Ads API v3 (allow-list required). Strong for subreddit/community targeting. API field names re-verified 2026-07-12 against multiple integrator schemas (Fivetran, dltHub, community MCPs). Use when working with Reddit ads specifically; loaded by the parent agentic-ads skill.
---

# Reddit Ads — Skill

Claude runs Reddit campaigns two ways — via browser automation of ads.reddit.com (works today, no API approval needed) or via the Reddit Ads API v3 (requires OAuth + allow-list approval). Both create in a paused state and require an explicit confirm before serving.

Reddit is a strong fit for community-intent targeting: you point ads at specific subreddits (r/CryptoCurrency, r/fintech, r/ethfinance, r/webdev) rather than guessing at interests. Subreddit targeting is the platform's core advantage over Meta/TikTok/X.

> **QA status (2026-07-12): 📝 Documented; browser path buildable today, API path buildable once your app is allow-listed.**
>
> This has not been run through a live create→delete cycle from this environment (no Reddit Ads credentials). API field names below were **cross-verified against three independent sources** during the 2026-07 research pass: the Fivetran `dbt_reddit_ads_source` schema (primary), the community-maintained Reddit Ads MCP server, and Reddit's own public Ads Help Center pages. The v3 API reference itself is behind an allow-list.
>
> The audit's earlier safety-critical concern about `is_enabled` vs `configured_status` is **resolved**: the correct field is **`configured_status`** with values `ACTIVE` / `PAUSED` / `ARCHIVED` / `DELETED`. `is_enabled` is not a valid Reddit field and would be silently ignored — an object POSTed with only `is_enabled: false` (no `configured_status`) defaults to `ACTIVE` and serves on save. Do not use `is_enabled` anywhere in Reddit API calls.

---

## Method 1: Browser automation (no API required)

Claude drives [ads.reddit.com](https://ads.reddit.com) directly in Chrome.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [Reddit Ads Manager](https://ads.reddit.com) in Chrome before starting
3. A Reddit Ads account (created free at ads.reddit.com; a payment method must be on file before a campaign can leave draft)

### What Claude does

Claude navigates Ads Manager, creates the campaign, sets objective, community/interest targeting, budget, and attaches a post. Before activating, it:
- Leaves the campaign **Paused** (won't serve)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

### Structure

```
Ad Account
└── Campaign      ← objective, schedule
    └── Ad Group  ← targeting, bid, budget (goal_value), placement
        └── Ad    ← the promoted post (creative + headline + CTA + destination)
```

**Budget lives on the ad group** (`goal_value` + `goal_type`), not the campaign — different from Meta and Google.

### Objectives

Reddit's objectives: **Brand Awareness**, **Reach**, **Traffic**, **Engagement**, **Video Views**, **Conversions**, **App Installs**, **Lead Generation** (beta), **Catalog Sales** (beta). For a cold audience with a landing page, use **Traffic**; for pure awareness with no click goal, use **Brand Awareness** or **Reach** (they're merged in the UI).

### Targeting (Reddit-specific)

| Type | What it does | Notes |
|---|---|---|
| **Communities** | Show ads in specific subreddits | The platform's strongest lever — target r/fintech, r/webdev directly |
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
| Expand Targeting on by default | Turn off for a controlled audience (API field: `expand_targeting: false`) |
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
Expand Targeting: off
```

### Delete after QA

```
Delete the paused Reddit campaign we just created — campaign name: [name]
```

Claude reads back the campaign name from the row before clicking Delete (ID-match guard), then opens the row overflow menu and confirms.

---

## Method 2: Reddit Ads API v3

Programmatic path. Requires OAuth **and** allow-list approval for write scopes.

- **Base URL:** `https://ads-api.reddit.com/api/v3/`
- **Docs:** `https://ads-api.reddit.com/docs/v3/` (allow-list-gated; not publicly readable)
- **Auth:** OAuth 2.0 Bearer token on every request (`Authorization: Bearer <token>`)
- **Rate limit:** ~1 request/second

### Prerequisites

1. **Register an app** at [reddit.com/prefs/apps](https://www.reddit.com/prefs/apps) → get `client_id` + `client_secret`.
2. **Request the Ads OAuth scopes** at authorization time: `adsread`, `adsedit`, `adsconversions`, `history`, `read`.
3. **Allow-list approval for write** — `adsedit` requires Reddit ads-team approval. Read-only (`adsread`) is available sooner; POST/PATCH returns 403 without the write allow-list on your app.
4. **OAuth flow:** authorize at `https://www.reddit.com/api/v1/authorize`, exchange the code at `https://www.reddit.com/api/v1/access_token`, refresh with the `refresh_token` grant when the access token expires (~1 hour).

Set credentials in your environment (read from a secret manager, not paste — history persists):

```
REDDIT_ADS_CLIENT_ID=...
REDDIT_ADS_CLIENT_SECRET=...
REDDIT_ADS_ACCESS_TOKEN=...
REDDIT_ADS_REFRESH_TOKEN=...
REDDIT_ADS_ACCOUNT_ID=...
```

Reddit access tokens expire in ~1 hour — automate refresh against `https://www.reddit.com/api/v1/access_token` with Basic auth = `client_id:client_secret` and `grant_type=refresh_token`, or any long job's second hour will 401.

### What Claude does via API

Create in order: **campaign → ad group → ad**. Targeting, bid, and **budget** live on the ad group. Everything is scoped to `ad_accounts/{account_id}`.

**Campaign** (start paused — `configured_status`):
```json
{
  "objective": "TRAFFIC",
  "configured_status": "PAUSED",
  "start_time": "2026-07-15T00:00:00Z"
}
```

**Ad group** (targeting + bid + budget):
```json
{
  "campaign_id": "<CAMPAIGN_ID>",
  "configured_status": "PAUSED",
  "bid_strategy": "MAXIMIZE_VOLUME",
  "goal_type": "DAILY_SPEND",
  "goal_value": 50000000,
  "expand_targeting": false,
  "targeting": {
    "geolocations": ["US"],
    "community_names": ["fintech", "webdev"]
  }
}
```

**Key API behaviors (verified 2026-07-12 against Fivetran schema + community MCP + Reddit help docs):**

- **`configured_status`** is the paused/active field. Values: `ACTIVE`, `PAUSED`, `ARCHIVED`, `DELETED`. `effective_status` is a read-only derived field — don't confuse them. `is_enabled` **is not a valid Reddit field** — if you POST with it, Reddit ignores it and your object defaults to ACTIVE.
- **Budget lives on the ad group**, not the campaign. Fields: `goal_type` (`DAILY_SPEND` / `LIFETIME_SPEND` / `IMPRESSIONS` / `CLICKS` / `CONVERSIONS` / `VIDEO_VIEWABLE_IMPRESSIONS` / `PERCENTAGE`) + `goal_value` (in **micros**: `$1 = 1,000,000`; `$50/day = 50000000`).
- **`expand_targeting: false`** is the cold-audience-expansion flag (Reddit's equivalent of Meta Advantage+ Audience). Default is `false` — leave it off explicitly for controlled audiences.
- **`bid_strategy`**: `MAXIMIZE_VOLUME` (default for cold audiences), `MANUAL_BIDDING` (post-learning, needs 500+ clicks + proven CPA).
- **Objectives**: `BRAND_AWARENESS`, `REACH`, `TRAFFIC`, `ENGAGEMENT` (fold to TRAFFIC — Reddit merged them), `VIDEO_VIEWS`, `CONVERSIONS`, `APP_INSTALLS`, `LEAD_GENERATION` (beta), `CATALOG_SALES` (beta). Exact SNAKE_CASE spellings are inferred from Reddit's UI/help — GET a live campaign to confirm before hardcoding in production automation.
- **Targeting IDs** (communities, interests, geos, devices) come from Reddit's targeting-search endpoints; resolve names to IDs before creating the ad group.

### Dry run mode

The Reddit Ads API has **no `validate_only` flag**. Two paths:
1. **Create with `configured_status: "PAUSED"`** on both campaign and ad group — real objects, real IDs, but not serving. **GET the object back to confirm** `configured_status: "PAUSED"` on the readback (safety verify). Delete after QA.
2. **Preview an Ad** endpoint — renders creative without publishing.

```
Create a Reddit campaign for [audience] — build it configured_status:PAUSED, GET the campaign back to verify the paused state, show me the payload, then wait for my explicit go.
```

### Delete after QA

Delete the ad group first, then the campaign — both are account-scoped in v3:

```
DELETE /api/v3/ad_accounts/{account_id}/ad_groups/{ad_group_id}
DELETE /api/v3/ad_accounts/{account_id}/campaigns/{campaign_id}
```

If DELETE isn't permitted for your access level, `PATCH configured_status: "ARCHIVED"` on each.

### Conversion tracking (Reddit Pixel + CAPI)

For `CONVERSIONS` or `CATALOG_SALES`, install the **Reddit Pixel** on the destination and/or send events server-side via the **Conversions API** (`docs/v3/capi-direct-integration`). Same shape as Meta pixel + CAPI: browser pixel plus server events recover conversions lost to blockers. Not needed for Traffic/Awareness QA.

### Regulated categories (finance / crypto)

Reddit's Financial & Cryptocurrency Products & Services policy: **restricted, not banned** — but **direct sales-rep management is required**, not self-serve. Advertiser must be vetted + licensed where legally required.

**Permitted (with rep):** exchanges, wallets, crypto cards, staking, lending, mining hw/sw, NFTs + marketplaces, CFDs, forex, brokerage.

**Prohibited outright:** single securities/tokens/coins (so promoting a specific stablecoin like "USDC" is banned), ICOs, IDOs, penny stocks, unlicensed banks/fintechs, payday loans, get-rich-quick, celebrity endorsements, liquidity pools, mixers, bots, DAOs without a centralized entity, **crypto loans**, leverage claims, guaranteed returns.

**For Eco (B2B stablecoin infra):** position as payments infrastructure / developer platform / integration / education. Do NOT promote a specific stablecoin ("Buy USDC") — that trips the single-token prohibition. Contact a Reddit rep for regulated-category approval before running.

Source: [business.reddithelp.com — financial/cryptocurrency policy](https://business.reddithelp.com/s/article/financial-cryptocurrency-products-and-services-policy)

### Key settings Claude enforces

- Campaign + ad group starts `configured_status: "PAUSED"` — verify by GET readback before any activation call
- Budget on the **ad group** via `goal_type` / `goal_value` in micros (`$1 = 1,000,000`)
- `expand_targeting: false` for cold audiences (leave default off)
- Regulated categories (finance/crypto): flag that a Reddit sales-rep is required before any live campaign; do not attempt to run these self-serve
- Never promote a specific token/coin/security by name (Reddit prohibits it outright)
