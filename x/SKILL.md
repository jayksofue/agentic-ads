---
name: agentic-ads-x
description: Deploy X (Twitter) ad campaigns via browser automation (ads.x.com) or the X Ads API v12. Save-draft-then-verify flow was live-QA'd 2026-07-09. Use when working with X/Twitter ads specifically; loaded by the parent agentic-ads skill.
---

# X Ads — Skill

Claude can run X campaigns two ways — via browser automation (works today, no API approval needed) or via the X Ads API (requires separate approval). Both support paused/draft mode.

> **QA status (2026-07-09): ✅ Method 1 (browser) verified end-to-end.** Ran the full cycle live on account `18ce54of0a1`: create campaign (Reach) → ad group targeting → Save draft → confirmed in the campaign list as `Draft` → deleted. Two behaviors were refined during QA (see the gotchas table): Save draft only persists from **Review and launch**, and the delete control is an **inline trash icon next to the campaign ID** on row hover. Method 2 (X Ads API) is still 📝 documented-only — no Ads API credentials configured, and access is approval-gated (typically weeks).

---

## Method 1: Browser automation (no API required)

Claude drives [ads.x.com](https://ads.x.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [X Ads Manager](https://ads.x.com) in Chrome before starting
3. Your X Ads account ID — found in the URL after logging in

### What Claude does

Claude navigates X Ads Manager, creates a campaign, sets objective, audience, budget, and attaches a tweet or creates a new promoted-only post. Before activating, it:
- Builds the wizard through to **Review and launch**, then clicks **Save draft** — the campaign persists in `Draft` state (verified 2026-07-09). There is no pre-launch Paused toggle; the mechanism is Save-draft-from-Review.
- Shows you a summary for review
- Activates only on your explicit confirmation (Publish from Review-and-launch)
- Can delete the draft campaign after QA if you ask

### Objective-specific behavior

| Objective | Ad Group: Optimization goal | Pay by | Ad level: unique fields |
|---|---|---|---|
| **Reach** | Impressions | Impressions (CPM) | Post text + Destination (Website/App) + media |
| **Engagements** | Engagements | Engagements (CPE) | Post text + Destination (Website/App) + media |
| **Website traffic** | Link clicks | Impressions (CPM) | Post text + **Website URL*** + **Headline*** + media (required for card ads) |
| **Video views** | Video views | Video views | Post text + Destination (Website/App) + media |
| **App installs** | App installs | App installs | Post text + app store link |
| **Sales** | Conversions | Impressions (CPM) | Post text + Website URL + conversion event |

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern: `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set` |
| "Optimize targeting" ON by default | Turn off for precise audience control — it's X's AI audience expansion (same as LinkedIn Audience Expansion) |
| Website traffic: extra required ad fields | Must supply Website URL + Headline + media — it creates a "website card" format, not a plain post |
| Website traffic: Dynamic Product Ads field | Appears at ad group level — toggle for e-commerce catalog ads; leave off for standard campaigns |
| 4 specific placements available | Home timeline, Search results, Profiles, Replies — all on by default; use "Choose specific placements" to exclude any |
| No third-party ad network | X ads are X-only; no equivalent of LinkedIn's LAN or Meta's Audience Network |
| Budget type defaults to daily | Switches to total budget if you specify a fixed amount |
| "Use existing post" picker | Shows Organic and Promoted-only tabs — can reuse any past ad creative with Clone & edit or Use as-is |
| Campaign name uses timestamp default | Use nativeSetter pattern to rename — keyboard input doesn't update React-controlled value |
| Delete draft campaign | Hover the campaign row → **inline red trash icon in the edit/duplicate/delete cluster next to the campaign ID** (not at the far right of the row) → "Delete this campaign?" modal → Delete. Verified 2026-07-09. |
| Save draft without ad content | Must be done from **Review and launch** → Save draft — works even with a publish-blocking ad error ("Post must have text or media"). **Clicking Save draft on the ad-group Details step does NOT persist a server campaign** — it only keeps a local unsaved draft (recovered via an "unsaved draft" modal on re-entry). Verified 2026-07-09. |
| Promote-only tweet not visible on timeline | Expected — it's ad-only by design |

### Campaign setup prompt

```
Set up an X campaign in Ads Manager — leave it as draft, don't launch.
Objective: Reach / Engagements / Website traffic / Video views / App installs / Sales
Post: [tweet URL or "write new promoted-only post: [copy here]"]
Audience: [keywords, interests, follower lookalikes of @handle, demographics, locations]
Budget: $[amount] total or daily
Start/end: [dates]
Optimize targeting: off
```

For **Website traffic**, also provide:
- Website URL (required)
- Headline (required)
- Creative image or video

### Delete after QA

```
Delete the draft X campaign we just created — campaign name: [name]
```

Claude hovers the campaign row, **reads back the campaign name from the row** to the operator, and clicks the red trash icon only after the operator confirms it matches the QA campaign name (the confirm modal is generic, so wrong-row-selected mistakes would not be caught by the modal alone).

---

## Method 2: X Ads API (v12, current as of 2026)

The X Ads API is free once approved but requires a separate Ads API application (distinct from general X API access). Apply via the Ads API onboarding page ([docs.x.com/x-ads-api/getting-started](https://docs.x.com/x-ads-api/getting-started)) — approval typically takes a few weeks.

**Base URL:** `https://ads-api.x.com/12/` (current — bump when X releases a new major).

### Prerequisites

Once approved:
- A developer app with Ads API access enabled
- `TWITTER_ADS_ACCOUNT_ID` (in the URL at ads.x.com)
- OAuth 1.0a signing keys: consumer key/secret + access token/secret

Set credentials in your environment (read from a secret manager, not paste them into a shell — history persists):

```
TWITTER_CONSUMER_KEY=...
TWITTER_CONSUMER_SECRET=...
TWITTER_ACCESS_TOKEN=...
TWITTER_ACCESS_TOKEN_SECRET=...
TWITTER_ADS_ACCOUNT_ID=...
```

Do **not** use the `twitter-ads` Python SDK — it was last released May 2022 against the retired v11 API and is unmaintained. Call the v12 REST endpoints directly with any OAuth 1.0a client (e.g. Python `requests-oauthlib`, Node `oauth-1.0a`).

### Campaign setup prompt

```
Create an X promoted post campaign:
- Account ID: [your account ID]
- Objective: ENGAGEMENTS / REACH / WEBSITE_CLICKS / FOLLOWERS / VIDEO_VIEWS / APP_INSTALLS
- Tweet to promote: [tweet URL or ID]
- Audience: [keywords, interests, follower lookalikes, demographics]
- Locations: [countries]
- Budget: $[amount] total or daily  (API expects micros — see note)
- Start/end: [dates]
- Dry run: yes/no
```

**Objective enum note:** valid v12 values are `ENGAGEMENTS` (plural, not `ENGAGEMENT`), `REACH` (not `AWARENESS` — that objective was renamed years ago), `WEBSITE_CLICKS`, `FOLLOWERS`, `VIDEO_VIEWS`, `APP_INSTALLS`, `APP_ENGAGEMENTS`, `PREROLL_VIEWS`. Older names (`AWARENESS`, `ENGAGEMENT` singular) will be rejected.

**Budget unit:** X Ads API expresses budgets in **micros of the local currency** — `total_budget_amount_local_micro` and `daily_budget_amount_local_micro`. For USD: `$1 = 1,000,000 micros`, so `$50/day = 50000000`. This mirrors Reddit's convention but differs from Meta (cents) and TikTok (whole dollars). A missing `_micro` suffix or a cents value is a 6-order-of-magnitude foot-gun — always sanity-check the number before submitting.

### Dry run mode

X Ads API has no native `validate_only` flag. Claude generates a full structured preview instead:

```
Show me the X campaign payload — don't submit yet.
```

Claude returns: full API request body (campaign + line item + targeting + creative), estimated audience size, and explicit confirmation prompt before any call is made.

### Delete after QA

```
Delete the X campaign with ID [campaign_id] — API call only, don't confirm.
```

Claude calls `DELETE https://ads-api.x.com/12/accounts/{account_id}/campaigns/{campaign_id}` (OAuth 1.0a-signed). Deleting a campaign cascades to its line items, but not to reserved tweets — the promoted-only tweet stays around and can be reused.

### Key settings Claude enforces

- **Scope `charge_by` to the objective**: `ENGAGEMENT` billing applies only to the Engagements objective. Reach, Website traffic, Sales, Video views use `IMPRESSION` (CPM); App installs uses `APP_INSTALLS`; Followers uses `FOLLOW`. Don't apply a blanket ENGAGEMENT charge — it's objective-scoped and the API rejects mismatched pairs.
- Campaign status: `PAUSED` until you explicitly confirm launch. Note: X's line-item `entity_status` uses `ACTIVE` / `PAUSED` / `DELETED`.
- `optimization_preference: DEFAULT` unless you override bid strategy.

### Algorithm note (January 2026 X source release)

Per the X algorithm breakdown (xai-org/x-algorithm, January 2026): threads are penalized by author diversity decay (2nd post = 55% of score, 3rd = 32.5%). For organic posts, single tweets outperform threads. For paid: promoted posts are injected directly and bypass feed scoring penalties — threads vs. single post doesn't affect paid performance.
