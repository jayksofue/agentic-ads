---
name: agentic-ads-tiktok
description: Deploy TikTok ad campaigns via browser automation (ads.tiktok.com) or the TikTok Marketing API v1.3. Documented not yet live-verified. Video-first — expect higher minimum budgets (~$50/day campaign). Use when working with TikTok ads specifically; loaded by the parent agentic-ads skill.
---

# TikTok Ads — Skill

Claude can run TikTok campaigns two ways — via browser automation (works today, no API approval needed) or via the TikTok Marketing API (requires a developer app and approval). Both support a paused state so you can review before spending.

> **QA status (2026-07-09): 📝 Documented, not yet verified end-to-end.** No TikTok Ads credentials are configured in this environment, and the Marketing API path requires a developer app that TikTok reviews before granting production access. Field names, budget units, and enums below are taken from the TikTok Marketing API v1.3 reference and its official SDK — treat them as unverified until a live run confirms them. The browser path (Method 1) needs the Claude for Chrome extension logged into TikTok Ads Manager.

TikTok skews younger and consumer; B2B fit is weaker than LinkedIn/Reddit, but its Marketing API is mature and it scales. Judge on cost per result, and expect higher minimum budgets than the other platforms (see gotchas).

---

## Method 1: Browser automation (no API required)

Claude drives [ads.tiktok.com](https://ads.tiktok.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [TikTok Ads Manager](https://ads.tiktok.com) in Chrome before starting (a TikTok for Business account is required)
3. Your Advertiser ID — shown in the Ads Manager URL / account switcher

### What Claude does

Claude navigates Ads Manager, creates the campaign, sets objective, ad group targeting + budget + placements, and attaches a video (or an existing TikTok post / Spark Ad). Before activating, it:
- Leaves the campaign / ad group **paused** (won't serve, won't spend)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

### Structure

```
Advertiser account
└── Campaign      ← objective, campaign-level budget (if CBO on)
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
| Automatic Targeting / Smart Targeting on by default | Turn off for a controlled audience (TikTok's equivalent of Meta Advantage+ / LinkedIn audience expansion) |
| Pangle + other placements on by default | Under Placements, switch from Automatic to **Select placement** and choose TikTok only if you don't want the Pangle audience network / News Feed apps |
| Minimum budget enforced | TikTok enforces higher minimums than other platforms — roughly **$50/day campaign** (CBO) and **$20/day ad group**. A budget below the minimum blocks the ad group |
| Creative required to publish | TikTok ads are video-first. You can promote an existing post (**Spark Ad**) or upload a video; a plain post won't publish without a video/image |
| Identity / TikTok account link | Spark Ads require authorizing the TikTok post via a video code / account link before it can be promoted |
| Delete draft/paused campaign | Campaigns list → select the campaign → status menu → **Delete** (or the row's delete/trash action) → confirm |

### Campaign setup prompt

```
Set up a TikTok campaign in Ads Manager — leave it paused, don't publish.
Objective: Traffic / Reach / Video Views / Engagement / Website Conversions
Audience: [geo, age, gender, languages, interests & behaviors]
Budget: $[amount] daily or lifetime  (mind TikTok's minimums)
Placements: TikTok only (no Pangle / News Feed apps)
Automatic targeting: off
Creative: [video to upload, or an existing post to run as a Spark Ad]
Destination URL: [landing page]
```

### Delete after QA

```
Delete the paused TikTok campaign we just created — campaign name: [name]
```

Claude selects the campaign in the list and deletes it via the status/row action.

---

## Method 2: TikTok Marketing API

Programmatic path — faster for bulk setup, no UI. Requires a developer app and TikTok's review.

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
  "operation_status": "DISABLE"
}
```

**Key API behaviors (from the v1.3 reference — confirm on a live run):**
- **Budget is in the account currency as a plain number (e.g., dollars), NOT cents or micros.** `"budget": 50` = $50. This is the opposite of Meta (cents) and Reddit (micros) — do not carry those conventions over.
- `budget_mode`: `BUDGET_MODE_DAY`, `BUDGET_MODE_TOTAL`, or `BUDGET_MODE_INFINITE`. With Campaign Budget Optimization on, only `BUDGET_MODE_DAY` is supported; for `RF_REACH` (reach & frequency) only `BUDGET_MODE_INFINITE`.
- `objective_type`: e.g. `REACH`, `TRAFFIC`, `VIDEO_VIEWS`, `ENGAGEMENT`, `LEAD_GENERATION`, `APP_PROMOTION`, `WEB_CONVERSIONS`, `PRODUCT_SALES` (and `RF_REACH` for reach & frequency). Confirm the exact enum for your API version — some values have been renamed across versions (older `LANDING_PAGE`/`APP`).
- **Paused-on-create caveat:** `operation_status` (`ENABLE` / `DISABLE`) is **allow-listed** in v1.3 — if your app is not allow-listed, TikTok **ignores it silently** and creates the object as `ENABLE`. So do not assume `DISABLE` took effect: re-read the object after creating, and if it is not disabled, disable it with `/campaign/status/update/` before adding creatives. Nothing serves until an ad group + approved ad exist, but treat this as the key safety check.
- Minimum budgets are enforced server-side (roughly $50/day campaign, $20/day ad group); a smaller value errors.

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

### Key settings Claude enforces

- Campaign/ad group created disabled (or in sandbox) until you explicitly confirm launch — and verified as disabled, given the `operation_status` allow-list caveat
- Budget always expressed in the account currency (dollars), never cents/micros
- Automatic targeting / audience expansion off unless you ask for it
- Placements restricted to TikTok unless you opt into Pangle / the news-feed apps
