---
name: agentic-ads-tiktok
description: Launch TikTok ad campaigns through ads.tiktok.com in the browser, or through TikTok's Marketing API for bulk work. Documented but not yet live-verified from this environment. TikTok is video-first (silent creative underperforms significantly) and enforces higher minimum budgets than the other platforms (roughly $50/day at the campaign level). Every campaign is created Paused; nothing goes live without an explicit "launch" from the user. Use this when launching TikTok ads specifically; loaded automatically by the parent agentic-ads skill.
---

# TikTok Ads — Skill

Two ways to launch TikTok campaigns:

- **Through the browser** — Claude drives TikTok Ads Manager directly in Chrome. Works for anyone with a TikTok for Business account.
- **Through TikTok's Marketing API** — faster for bulk work. Requires a TikTok developer app that goes through TikTok's review process.

Both paths keep the campaign in a Paused state until you say launch.

> **Status: 📝 Documented, not yet verified end-to-end.** The API-side details below have not been end-to-end tested from this environment — no TikTok Ads credentials are configured here, and the Marketing API path requires a developer app that TikTok reviews before granting production access. Setting names, budget formats, and available options below come from TikTok's Marketing API v1.3 reference and their official SDK; treat them as accurate but confirm on a live run. The browser path also needs the Claude for Chrome extension logged into TikTok Ads Manager.

**Who TikTok is for.** The audience skews younger and consumer — B2B usually works better on LinkedIn or Reddit. TikTok is video-first (silent creative significantly underperforms), and the platform enforces **higher minimum budgets** than the others (roughly $50/day at the campaign level — see the gotchas table below). If you're spending less than about $1,500/month, another platform will likely give you better results.

---

## Method 1: Through the browser (no API setup needed)

Claude drives [ads.tiktok.com](https://ads.tiktok.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [TikTok Ads Manager](https://ads.tiktok.com) in Chrome before starting (a TikTok for Business account is required)
3. Your Advertiser ID — shown in the Ads Manager URL / account switcher
4. **Payment method on file** (Payments → Add). Activation is blocked without a funding source. TikTok also requires a **business verification** step before some categories (finance, crypto) can serve — do this in advance if launching regulated categories.

### What Claude does

Claude navigates Ads Manager, creates the campaign, sets objective, ad group targeting + budget + placements, and attaches a video (or an existing TikTok post / Spark Ad). Before activating, it:
- Leaves the campaign / ad group **paused** (won't serve, won't spend)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

### Structure

```
Advertiser account
└── Campaign      ← objective, campaign-level budget (if Campaign Budget Optimization / CBO is on)
    └── Ad Group  ← targeting, bid, schedule, placements, optimization goal, ad-group budget
        └── Ad    ← the creative (video + text + CTA + destination)
```

Targeting lives at the **ad group** level (same as Meta's ad set).

### Objectives

TikTok's ad-group/campaign objectives: **Reach**, **Traffic**, **Video Views**, **Engagement** (community interaction), **Lead Generation**, **App Promotion**, **Website Conversions**, **Product Sales**. For a cold audience with a landing page, use **Traffic**; for awareness, **Reach**.

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern: `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set` |
| Smart Targeting (audience + interest/behavior expansion) | TikTok's equivalent of Meta Advantage+ / LinkedIn audience expansion. **Fields split into two booleans** (both opt-in, default off): `smart_audience_enabled` and `smart_interest_behavior_enabled`. The older `auto_targeting_enabled` field was **deprecated June 2024** — cannot be enabled anymore; ignore it. Keep both smart-targeting fields off for cold-audience runs |
| Pangle + other placements on by default | Under Placements, switch from Automatic to **Select placement** and choose TikTok only if you don't want the Pangle audience network / News Feed apps |
| Minimum budget enforced | TikTok enforces higher minimums than the other platforms — roughly **$50/day at the campaign level** (when Campaign Budget Optimization / CBO is on) and **$20/day at the ad-group level** (when the budget lives there instead). Anything below the minimum gets rejected. |
| Creative required to publish | TikTok ads are video-first. You can promote an existing post (**Spark Ad**) or upload a video; a plain post won't publish without a video/image |
| Identity / TikTok account link | Spark Ads require authorizing the TikTok post via a video code / account link before it can be promoted |
| Delete draft/paused campaign | Campaigns list → select the campaign → status menu → **Delete** (or the row's delete/trash action) → confirm |

### Creative specs (TikTok is video-first — assets get rejected at review otherwise)

| Spec | Value |
|---|---|
| **Aspect ratio** | 9:16 (vertical) preferred; 1:1 accepted; 16:9 works but performs poorly in-feed |
| **Resolution** | 720×1280 minimum; **1080×1920 recommended** for 9:16 |
| **Duration** | 5–60 seconds (in-feed ads); TopView up to 60s |
| **File format** | MP4, MOV, MPEG, 3GP, AVI |
| **File size** | ≤500 MB |
| **Bitrate** | ≥516 kbps |
| **Safe zones** | Keep critical text/logos out of the top ~150px (username/caption overlay) and bottom ~350px (CTA + music strip on TikTok's in-feed layout). Use TikTok's Video Editor "safe zone" preview to check. |
| **Audio** | Sound is expected — TikTok's audience watches with sound on far more than Meta/IG. Silent-optimized creative underperforms here. |
| **Spark Ads** | Use an existing TikTok post's video code (Creator Marketplace or manual auth) — inherits organic engagement + comments |

### Campaign setup prompt

```
Set up a TikTok campaign in Ads Manager — leave it paused, don't publish.
Objective: Traffic / Reach / Video Views / Engagement / Website Conversions
Audience: [geo, age, gender, languages, interests & behaviors]
Budget: $[amount] daily or lifetime  (mind TikTok's minimums)
Placements: TikTok only (no Pangle / News Feed apps)
Automatic targeting: off
Creative: [video to upload — 9:16, 1080×1920, ≤500 MB, MP4 — or an existing post to run as a Spark Ad]
Destination URL: [landing page]
```

### Delete after QA

```
Delete the paused TikTok campaign we just created — campaign name: [name]
```

Claude selects the campaign in the list and deletes it via the status/row action.

---

## Method 2: TikTok's Marketing API

For bulk work or repeatable setups, you can drive TikTok through their Marketing API instead of the browser. Requires a TikTok developer app that has to go through TikTok's review before it can create real production campaigns (there's a sandbox for testing while you wait).

- **Base URL:** `https://business-api.tiktok.com/open_api/v1.3/`
- **Docs:** TikTok for Business API portal (`business-api.tiktok.com/portal/docs`)
- **Official SDKs:** `tiktok-business-api-sdk` (Python and JS)
- **Auth:** OAuth 2.0 — exchange the auth code for a long-lived access token; pass it in the `Access-Token` header on every request

### Prerequisites

1. Create a developer app in the TikTok for Business / Marketing API portal and get the App ID + Secret.
2. Complete TikTok's app review to get production access (sandbox is available for testing without review).
3. Authorize the advertiser account via OAuth to obtain the `Access-Token` and the `advertiser_id`.

Set credentials in your environment:

```
TIKTOK_APP_ID=...
TIKTOK_APP_SECRET=...
TIKTOK_ACCESS_TOKEN=...
TIKTOK_ADVERTISER_ID=...
```

### What Claude does via API

Create in order: **campaign → ad group → ad**. Targeting, bid, schedule, and placements live on the ad group.

**Campaign** (`POST /campaign/create/`):
```json
{
  "advertiser_id": "<ADVERTISER_ID>",
  "campaign_name": "QA_Traffic_Jul2026",
  "objective_type": "TRAFFIC",
  "budget_mode": "BUDGET_MODE_DAY",
  "budget": 50,
  "buying_type": "AUCTION",
  "operation_status": "DISABLE"
}
```

**Key API behaviors (verified 2026-07-12 against TikTok v1.3 docs):**
- **Budget field is `budget` — float in account currency, NOT cents, NOT micros.** For USD: `"budget": 50` = $50 (precision 0.01). This is the odd one out — Meta uses cents, Reddit/X/Google use micros. Do not carry those conventions over.
- `budget_mode`: `BUDGET_MODE_DAY`, `BUDGET_MODE_DYNAMIC_DAILY_BUDGET`, `BUDGET_MODE_TOTAL`, `BUDGET_MODE_INFINITE` (forced for `RF_REACH`).
- **`buying_type`**: `AUCTION` (default; auction-based) or `RESERVATION` (Reach & Frequency reservation buys). Include explicitly — defaults can shift across API versions.
- `objective_type`: `TRAFFIC`, `REACH`, `RF_REACH` (guaranteed reach), `ENGAGEMENT`, `VIDEO_VIEWS`, `WEB_CONVERSIONS`, `LEAD_GENERATION`, `APP_PROMOTION`, `PRODUCT_SALES`. All confirmed against v1.3 primary docs 2026-07-12.
- **Reach & Frequency uses `RF_REACH` as an objective_type** paired with `buying_type: "RESERVATION"` + `budget_mode: "BUDGET_MODE_INFINITE"` + `/adgroup/rf/create/` endpoint. This is a documented API path (confirmed), not the older "buying_type-only" pattern I documented earlier — updated 2026-07-12.
- **Paused-on-create**: `operation_status: "DISABLE"` at create time (default is `"ENABLE"`). **Caveat for RF campaigns:** `DISABLE` cannot be passed on RF campaigns — you must pass `ENABLE` or omit. For auction campaigns, always create `DISABLE`, GET-verify, then flip `ENABLE`.
- **Bid fields moved to ad group in v1.3.** `bid_type`, `deep_bid_type`, `roas_bid`, `optimize_goal` are **deprecated at the campaign level** — they're now ad-group fields only. If you set them on a campaign, they're ignored.
- Minimum budgets (per TikTok's budget-verification-ratio table, USD): ~**$50/day campaign / $20/day ad group** for auction; **$10/day** floor for PRODUCT_SALES/STORE. Below the floor, the API returns an error.

### Token refresh

TikTok access tokens are long-lived by default (~365 days) but *do* eventually expire. Use `/oauth2/refresh_token/` to exchange a refresh token for a new access token; store both in your secret manager, not shell env. Rotate access tokens if any leak or if the app's OAuth scopes change.

### Dry run mode

The TikTok Marketing API has **no `validate_only` flag**. To preview without spending:
1. Use the **sandbox** advertiser (developer portal) — full create/read/delete with no real spend or review.
2. Or create in production with the object **disabled** and no active ad, then delete after QA.

```
Create a TikTok campaign for [audience] — build it disabled, show me the payload, don't enable it.
```

Claude returns the full request bodies (campaign + ad group + ad) and an explicit confirm prompt before any call.

### Delete after QA

Use the status-update endpoints with `operation_status: "DELETE"` (delete the ad group first, then the campaign):
```
POST /adgroup/status/update/   { advertiser_id, adgroup_ids: [...], operation_status: "DELETE" }
POST /campaign/status/update/  { advertiser_id, campaign_ids: [...], operation_status: "DELETE" }
```
The same endpoints take `ENABLE` / `DISABLE` to pause or resume.

### Regulated categories (finance / crypto)

TikTok's Financial Services policy is one of the strictest in the industry.

**Cryptocurrency, exchanges, wallets, NFT trading, crypto advisory are PROHIBITED in the US** — the policy explicitly lists these as not allowed for US-based ads. Some Latin American markets (Argentina, Chile, Colombia, Ecuador, Mexico, Peru, Uruguay) may permit crypto ads with local license + 18+ gating + TikTok Sales Rep approval + disclaimers.

**Self-serve is not available for regulated finance** — a Sales Rep application is required for any restricted-category advertiser.

**For a stablecoin infra brand** (Eco-style, B2B, developer-focused): **do not target US audiences on TikTok**. Other geos require Rep + Verified Business Account + 18+ gating + local license + disclaimers. Positioning must be strictly B2B/infra/education, never consumer-facing "buy/hold/yield."

**`special_industries` field** (`HOUSING`, `EMPLOYMENT`, `CREDIT`) is US/CA **anti-discrimination law compliance**, not crypto/finance. It restricts detailed-targeting features for those verticals in the US/CA under anti-discrimination law — **not the same as Meta's crypto authorization**. Once selected on a campaign, the value **cannot be changed or re-enabled**, only removed.

Source: [ads.tiktok.com — Financial Services policy](https://ads.tiktok.com/help/article/tiktok-ads-policy-financial-services)

### Key settings Claude enforces

- Auction campaigns: create with `operation_status: "DISABLE"`, GET-verify, then flip `"ENABLE"`. RF campaigns can't be created disabled (documented).
- Budget: `budget` (float, account currency) — never cents or micros
- Smart Targeting: `smart_audience_enabled` and `smart_interest_behavior_enabled` both left `false` for cold audiences. Ignore the deprecated `auto_targeting_enabled`.
- Placements: TikTok only by default; opt in to Pangle / News Feed apps explicitly if wanted
- **Crypto/stablecoin advertisers in the US**: block campaign create — the platform prohibits it. For allowed geos, require Sales Rep + Verified Business Account before any create call.
