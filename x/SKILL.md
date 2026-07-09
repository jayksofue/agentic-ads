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
- Sets campaign status to **Paused** (won't serve, won't spend)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

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

Claude hovers the campaign row, clicks the red trash icon, and confirms Delete.

---

## Method 2: X Ads API

The X Ads API is free once approved but requires a separate application from general X API access. Apply at [ads.x.com/help](https://ads.x.com/help). Approval typically takes a few weeks.

### Prerequisites

Once approved:
- A developer app with Ads API (Standard Access) enabled
- `TWITTER_ADS_ACCOUNT_ID` (in the URL at ads.x.com)
- Consumer key/secret + access token/secret

Set credentials in your environment:

```
TWITTER_CONSUMER_KEY=...
TWITTER_CONSUMER_SECRET=...
TWITTER_ACCESS_TOKEN=...
TWITTER_ACCESS_TOKEN_SECRET=...
TWITTER_ADS_ACCOUNT_ID=...
```

Optional: install the SDK:

```bash
pip install twitter-ads
```

### Campaign setup prompt

```
Create an X promoted post campaign:
- Account ID: [your account ID]
- Objective: ENGAGEMENT / AWARENESS / WEBSITE_CLICKS / FOLLOWERS
- Tweet to promote: [tweet URL or ID]
- Audience: [keywords, interests, follower lookalikes, demographics]
- Locations: [countries]
- Budget: $[amount] total or daily
- Start/end: [dates]
- Dry run: yes/no
```

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

Claude calls `DELETE /1/accounts/{account_id}/campaigns/{campaign_id}`.

### Key settings Claude enforces

- `charge_by: ENGAGEMENT` — pay per engagement, not impression
- Campaign status: `PAUSED` until you explicitly confirm launch
- `optimization_preference: DEFAULT` unless you override bid strategy

### Algorithm note (May 2026 X source release)

Per the X algorithm breakdown: threads are penalized by author diversity decay (2nd post = 55% of score, 3rd = 32.5%). For organic posts, single tweets outperform threads. For paid: promoted posts are injected directly and bypass feed scoring penalties — threads vs. single post doesn't affect paid performance.
