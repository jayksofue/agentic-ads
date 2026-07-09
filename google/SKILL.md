# Google Ads — Skill

Claude can run Google campaigns two ways — via browser automation (works today, no API approval needed) or via the Google Ads API (requires a developer token). Both support draft/paused mode.

> **QA status (2026-07-09): ✅ Create/draft flow verified; ⚠️ draft deletion is a known gap.** Ran live on the active **Eco 2025** account (237-467-3790): created a Search campaign via "Create a campaign without guidance" → named it (nativeSetter) → the documented "Save as a campaign draft?" modal appeared → Saved → the draft persisted and was confirmed in the campaigns table under "Drafts in progress" with a real campaign ID. Nothing served or spent.
>
> **Important cleanup finding:** once a "draft in progress" is **saved**, it has **no delete control in the campaigns UI** — no inline trash on the row, no actions column, the bulk **Edit is disabled** for drafts, the builder has no delete button, and auto-save means clicking the close (X) no longer prompts the Save/Discard modal. So for QA, **click Discard in the exit modal instead of Save** (the documented cleanup) — do NOT Save the draft. To remove an already-saved draft, use Google Ads Editor / the API, or finish+publish it as a PAUSED campaign (real campaigns do expose a Remove status control) and then remove it.
>
> First attempt hit a different account ("Pump The Beat") that was suspended/canceled, which disables "New campaign" entirely — switch accounts via the account selector. Method 2 (Google Ads API) remains unverified: no developer token configured, and Standard Access is approval-gated.

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

### Campaign types and what changes per type

| Campaign type | Unique wizard steps | Default bid strategy | Key unique fields |
|---|---|---|---|
| **Search** | Bidding → Campaign settings → AI Max → Keyword & asset gen → Keywords & ads → Budget → Review | Maximize Conversions | Keywords, ad copy (headlines + descriptions), Final URL |
| **Video** | Subtype selection → Bidding → Campaign settings → Budget → Review | Maximize Conversions or CPV | YouTube video URL, audience targeting, ad format per subtype |
| **Performance Max** | Asset groups → Audience signals → Budget → Review | Maximize Conversions | Asset group (images + logos + headlines + descriptions + videos + final URL) — no keywords |
| **Display** | Bidding → Targeting → Budget → Review | Maximize Conversions | Image creative, responsive display ad assets |
| **Demand Gen** | Bidding → Targeting → Budget → Review | Maximize Conversions | Image/video creative, runs on YouTube + Gmail + Discover |

### Video campaign subtypes (unique to Video)

After selecting Video campaign type, a **"Select a campaign subtype"** step appears:

| Subtype | Ad formats | How you pay |
|---|---|---|
| **Video views** (default) | Skippable in-stream, in-feed, Shorts | Per view (TrueView — pay when viewer chooses to watch) |
| **Efficient reach** | Bumper, skippable in-stream, in-feed, Shorts | CPM |
| **Non-skippable reach** | Bumper, standard non-skippable, 30s non-skippable | CPM |
| **Target frequency** | Bumper, skippable/non-skippable in-stream, in-feed, Shorts | CPM |
| **Drive conversions** | Skippable in-stream + conversion-oriented formats | CPM |
| **Ad sequence** | Mix of skippable, non-skippable, or bumper in sequence | CPM |
| **Audio reach** | Audio ads on YouTube | CPM |
| **YouTube subscriptions and engagements** (NEW) | Video engagement formats | Engagement-based |

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| Smart campaigns prompt on account creation | Dismisses — uses Expert Mode |
| "Optimize campaign" suggestions | Skips all auto-apply recommendations |
| Display Network checked by default on Search | Unchecks under Campaign settings → Networks |
| Search Network partners on by default | Unchecks unless you want it |
| **AI Max: Text customization ON by default** | Turns off under AI Max step — otherwise Google rewrites your ad copy dynamically from your website |
| **AI Max: Final URL expansion ON by default** | Turns off under AI Max step — otherwise Google can redirect clicks to different pages on your site |
| Broad match keywords by default | Wraps terms in `"quotes"` for phrase or `[brackets]` for exact unless you specify |
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern same as LinkedIn |
| Budget defaults to Average daily | Switches to Campaign total if you specify a fixed amount |
| "Save as a campaign draft?" modal on close | Three options: Cancel (stay on form), Discard (delete), Save (save as draft) — Claude clicks Discard after QA |
| Customer acquisition: bids equally for new and existing by default | Leave as-is unless you explicitly want new-customer-only targeting |
| Video subtype selection required | Must select correct subtype — default is Video views (TrueView); wrong subtype changes ad format and billing |
| Conversion goals pre-populated from account defaults | Shows Sign-ups and Submit lead forms with warning icons — safe to proceed without changing these for QA |

### Campaign setup prompt

```
Set up a Google Ads campaign in the UI — leave it Paused, don't publish.
Campaign type: Search / Display / Video / Performance Max
Goal: Awareness / Leads / Website traffic / Conversions
Keywords or audience: [keyword list or audience description]
Locations: [countries or cities]
Budget: $[amount] daily or total
Bidding: Maximize clicks / Target CPA $[amount] / Manual CPC
Ad copy: [headline 1, headline 2, headline 3 / descriptions — or ask Claude to generate]
Final URL: [your landing page]
AI Max: off (keeps exact ad copy and URL control)
Display Network: off (Search campaigns only)
```

For **Video** campaigns, also provide:
- Video subtype (Video views / Efficient reach / Non-skippable reach / etc.)
- YouTube video URL
- Audience targeting (interests, keywords, demographics, custom segments)

For **Performance Max** campaigns, provide:
- Asset group: 3–5 headlines, 2 descriptions, 1+ image (1.91:1), 1+ square image (1:1), logo, final URL
- Audience signals (optional — tells Google's AI who to start targeting)

### Delete after QA

For a **published** (paused) campaign, Claude navigates to it, selects it, and sets status to Removed via the status control — real campaigns expose this.

For a **draft in progress** (built but never published), there is no delete in the campaigns UI (verified 2026-07-09). The correct move is to **not save it**: click **Discard** in the "Save as a campaign draft?" modal on exit. If a draft was already saved, remove it with Google Ads Editor or the API, or finish+publish it paused and then remove it.

```
For QA: build the campaign, review it, then Discard on exit (don't Save the draft).
```

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
