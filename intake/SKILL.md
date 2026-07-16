---
name: agentic-ads-intake
description: Guided walkthrough for launching an ad campaign — asks 10 core questions, applies platform best practices, blocks anything that would fail policy or waste money before you deploy. Loads the right platform sub-skill (LinkedIn/Meta/X/Google/Reddit/TikTok) based on the answers. This is the entry point for the agentic-ads skill.
---

# Intake — Guided Ad-Campaign Walkthrough

Ask 10 questions. Apply the constraints in [routing-tables.md](./routing-tables.md). Load the platform sub-skill(s) that fit. Block the deploy if any hard constraint fails.

**Design principle: prescriptive on safety, advisory on strategy.** The intake enforces what would fail or waste money (no debate); it surfaces trade-offs where the user actually has a choice. It never tells the operator which platform is "best" without knowing their inputs — but it will disqualify platforms whose policy/budget/pixel/creative constraints their inputs would break.

---

## The 10-question walkthrough

Ask these in order. Each question has a **why-it-matters** line, an **example**, and a **default the user can accept with one word**. If the user says "use default" or "same as last" or "your call," accept and move on. If the user pastes a full brief up front (>200 chars covering multiple questions), skip to the confirmation summary and fill missing gaps only.

### 1. What are you promoting? (brand + product in one line)

> **Why:** anchors copy generation + detects regulated categories (finance / crypto / health / employment / housing / gambling / politics / alcohol)
> **Example:** "our SaaS invoicing tool for freelancers" · "Nike Pegasus 41 running shoes" · "our webinar on incident response for SREs" · "our B2B stablecoin payments infrastructure"

Detect regulated category from this answer. Set `regulated_category` internally to one of: `financial_services` | `crypto` | `health` | `employment` | `housing` | `credit` | `gambling` | `politics` | `alcohol` | `none`. Confirm with the user if the detection matters ("this looks like a financial-services ad — is that right? affects which platforms can run it and what declarations they need").

### 2. What's the goal?

> **Why:** determines the objective enum on every platform
> **Options (present as list; user picks one):** `awareness` · `traffic` · `engagement` · `video_views` · `leads` · `conversions` · `app_installs` · `sales` · `followers` (X-only)
> **Default:** `traffic` for a cold audience with a landing page; `awareness` if no landing page

If user picks `conversions` / `sales` / `leads`, ask Q2a: **"Do you have a pixel/conversion event firing with 30+ recent events?"** If no → warn and offer to fall back to `traffic` (see the routing table: Meta/Google/TikTok/Reddit all need pixel volume to optimize for conversions; without it they underdeliver).

### 3. Ideal customer in one paragraph

> **Why:** feeds targeting + copy generation
> **Example:** "US CTOs and heads of engineering at fintech startups, Series A–C, actively integrating payments infrastructure" · "US women 25–34 who follow running influencers and shop Nike/Lululemon"
> **Default:** ask — this is the highest-leverage input

Extract from the paragraph: demographics (age, gender, geo, language), professional attributes (role, industry, seniority, company size — only for B2B), interests/behaviors, purchase intent. Feed each dimension into the per-platform targeting router in [routing-tables.md](./routing-tables.md).

### 4. Where do they live? (countries + optional cities)

> **Why:** geo targeting + regulated-category geo eligibility check
> **Example:** "US, UK, Singapore" · "US only" · "US + Canada + EU"
> **Default:** infer from Q3 (if audience paragraph mentioned geos, use those)

For regulated categories, cross-check against the platform's geo allowlist for that category (in routing-tables.md). If the target geo isn't on the platform's allowlist for the category, disqualify the platform for that combination.

### 5. Total budget + shape

> **Why:** sets budget mode + minimum feasibility
> **Options:** `$X lifetime over N days` · `$X/day always-on` · `$X total for a fixed campaign`
> **Example:** "$500 lifetime over 30 days" · "$50/day for a month" · "$1500 monthly recurring"
> **Default:** ask — no reasonable default here

Immediately run **Feasibility check A: platform minimum**. Convert lifetime to $/day (lifetime ÷ days), then compare against each platform's practical minimum (from routing-tables.md). Disqualify any platform where $/day falls below its practical minimum, with the explicit reason.

### 6. Timing

> **Why:** start/end dates + ad-review SLA guardrails
> **Options:** `ASAP` · `starts [date]` · `runs [start] to [end]` · `always-on until paused`
> **Default:** `ASAP` (start tomorrow to allow ad review)

Cross-check against ad-review SLAs in the root SKILL.md. If the user says "campaign has to run by [date]" and the ad-review SLA (especially for regulated categories) doesn't leave time, warn: "regulated crypto/finance ads take 3–5 business days for review on Google, days–weeks on Meta/LinkedIn/Reddit/TikTok — your [date] deadline is tight; consider Traffic objective on a non-regulated platform, or start now."

### 7. What creative do you have?

> **Why:** determines ad format + platform fit
> **Options:** `video 9:16` (TikTok-native, works on Reels/Shorts) · `video 16:9` (YouTube, LinkedIn) · `video 1:1` (Meta Feed, X) · `single image` · `carousel` · `document/PDF` (LinkedIn-only) · `existing post to promote` · `nothing yet — help me plan`
> **Default:** ask — creative gates platform choice

If the user has no creative and no time to make one, offer: (a) copy-only formats (X promoted post, Reddit text post, Google Search text ads); (b) Claude generates copy + you find one image (Meta single-image, LinkedIn single-image); (c) delay campaign until creative is ready. Route to [copy/SKILL.md](../copy/SKILL.md) for copy generation.

Run **Feasibility check B: creative-format match**. If user picks a platform where their creative doesn't fit (e.g. Document ad → LinkedIn only; 9:16 video → TikTok/Reels-native, works on Meta but underperforms on LinkedIn feed), warn or disqualify.

### 8. Where should clicks go? (destination)

> **Why:** landing URL + conversion tracking setup
> **Options:** `landing page URL` · `app store link` · `existing lead form` · `phone number` · `nothing — awareness only`
> **Example:** "eco.com/build" · "apps.apple.com/…"
> **Default:** ask if goal is `traffic`/`conversions`/`leads`/`app_installs`; skip if goal is `awareness`/`video_views`

If `conversions` goal but no pixel installed on destination → fail loud: "we can't optimize for conversions on a page with no pixel. Options: (a) install the platform's pixel first, (b) switch goal to Traffic, (c) skip." Do not silently fall back.

### 9. Tone / brand voice

> **Why:** copy generation constraints; anti-slop
> **Options:** `confident + technical` · `friendly + accessible` · `authoritative + formal` · `playful + irreverent` · `provide examples of past copy` · `same as our website`
> **Default:** `confident + technical` for B2B; `friendly + accessible` for consumer; infer from Q1

Pass tone + Q10 don'ts into [copy/SKILL.md](../copy/SKILL.md) for copy generation.

### 10. Anything to avoid in the ad?

> **Why:** brand-safety guardrails + slop rules
> **Example:** "no em-dashes, no 'seamless/innovative/cutting-edge', no 'game-changer'" · "don't mention specific competitors" · "don't imply guaranteed returns"
> **Default:** apply the standard anti-slop checklist from [copy/SKILL.md](../copy/SKILL.md)

Note: for regulated categories (crypto, finance, health), some phrases are policy-required to avoid — the intake enforces those regardless of user preference (e.g. no "guaranteed returns" on crypto ads across every platform).

---

## Platform-specific follow-up questions (only ask if relevant)

After the 10 core questions, ask 1–2 platform-specific questions only if the user selected a platform that needs them:

| If user picked | Also ask |
|---|---|
| **LinkedIn** | Job function + seniority + company size band (LinkedIn's B2B targeting is unique) |
| **Meta** | Custom-audience/lookalike sources — only if pixel data exists (from Q8) |
| **X** | Follower-lookalike @handles + keyword targeting (X's unique interest model) |
| **Google Search** | Seed keywords (2–5) and match type preference (phrase/exact) |
| **Google Video / Demand Gen** | YouTube video URL (asset must exist on the channel) |
| **Google PMax** | Asset group source (existing landing page URL is enough — Google infers) |
| **Reddit** | Which subreddits (Reddit's core lever — always ask, don't auto-pick) |
| **TikTok** | Spark Ad or fresh upload? If Spark, TikTok post URL + Creator Marketplace auth code |

If the user chose multiple platforms in Q1a (see multi-platform mode below), ask each platform's follow-up separately.

---

## Multi-platform mode

**Supported:** launching the same brief across multiple platforms as **independent, parallel campaigns**. Each campaign is its own object on its own platform — same creative + same audience intent, but separate campaign IDs, separate budgets, separate reporting.

**Not supported (v1):** cross-platform campaigns that share a budget or report as one entity. Every platform's ad account is independent; there is no unified "spend $500 across LinkedIn + Meta with automatic allocation" mode in this skill.

If the user says "run this on LinkedIn and Meta," treat as two separate campaign creates. If they say "run on all platforms that fit," suggest the ones that pass feasibility and let them pick.

---

## Feasibility check layer

Run **before any create call**. Fail loud with actionable fix — don't just warn.

### A. Budget × duration hits platform minimum

Convert to $/day (lifetime ÷ days). Compare each candidate platform's practical minimum:

| Platform | Practical daily minimum |
|---|---|
| Meta | ~$20/day (needs ~50 events/week to exit learning phase) |
| LinkedIn | $25/day Sponsored Content (or $10/day for Text Ads specifically) |
| X | ~$5–10/day for engagement/reach; **~$50/day for conversion objectives** |
| Google Search | 10× expected CPC (~$10–50/day depending on keyword competitiveness) |
| Reddit | $50–100/day for the algo to learn (technical min is $5/day but starves at that level) |
| TikTok | $50/day campaign / $20/day ad group (hard-enforced by the API) |

If below → **block with the exact fix**: "$100 lifetime × 30 days = $3.30/day. TikTok requires ~$50/day; Meta needs ~$20/day. Raise to $600 lifetime over 30 days, or switch to Traffic + a lower-minimum platform like X or Reddit."

### B. Conversion goal has pixel with volume

If goal ∈ {`conversions`, `sales`, `leads` on Meta}, check:
1. Pixel installed on destination? (user reports)
2. 30+ events fired in the last 30 days on the target event?

If either is no → **block**: "Conversion optimization needs 30+ recent events to work. Options: (a) install the pixel + drive 30+ events via Traffic first, (b) switch this campaign to Traffic and re-run as Conversions once you have volume, (c) skip."

### C. Regulated category has required declaration/approval

For each platform × regulated-category combination in [routing-tables.md](./routing-tables.md):

| Category | Meta | LinkedIn | X | Google | Reddit | TikTok |
|---|---|---|---|---|---|---|
| Cryptocurrency (any) | Auth & Verifications approval (days–weeks) | Sales-rep whitelist (days–weeks) | Category cert (days) | Crypto cert (3–5 biz days) | Sales rep (multi-week) | **US: prohibited.** LatAm only, Sales Rep |
| Financial services | `special_ad_categories: [FINANCIAL_PRODUCTS_SERVICES]` (India-specific) OR Auth & Verifications | Sales-rep whitelist | Financial services cert | Financial products cert | Sales rep | Sales Rep + Verified Business Account |
| Employment / Housing / Credit (US anti-discrimination) | `special_ad_categories: [EMPLOYMENT / HOUSING / CREDIT]` | Automatically flagged via account | Similar restrictions apply | Standard | Automatic | `special_industries` field |

If the operator hasn't completed the required approval for the platforms they picked → **block with the fix**: "Meta requires Business Suite → Authorizations & Verifications for crypto. Path: (a) start the approval now (days–weeks), (b) route to X (Cryptocurrency cert is fastest, few days), (c) route to Google (Crypto cert 3–5 biz days), (d) run browser path on Reddit (needs a sales rep before publishing but you can build the draft)."

### D. Creative spec matches format

Compare the user's creative dimensions to each candidate platform's format requirements (aspect ratio, resolution, duration, file size). If creative doesn't fit → warn if it's a soft mismatch (16:9 on Meta Feed underperforms 1:1), block if it's a hard mismatch (100 MB image → over Meta's 30 MB limit).

### E. Ad account access level

Ask before build: "Are you an Admin/Advertiser on the [platform] ad account, not just a Viewer?" Viewers cannot create campaigns. If unsure, route them to check under Account Settings on the platform.

---

## The confirmation summary

Before any create call, echo everything back to the operator. Every default the skill applied is listed explicitly — no hidden decisions.

Template:

```
Ready to build. Confirming your setup:

Platform:            [LinkedIn]
Objective:           [WEBSITE_VISIT / traffic]  ← from your goal "traffic"
Audience:            [US CTOs, engineering leads at fintech Series A–C]
                     LinkedIn targeting: Function=Engineering|IT + Seniority=Director+VP+CXO
                     + Industry=Financial Services|Software + Company size=51–1000
Estimated reach:     [~180K members]  ← within LinkedIn's 50–400K sweet spot ✓
Geo:                 [US, UK, Singapore]
Budget:              [$500 lifetime over 30 days = ~$16.70/day]
                     ⚠ Below LinkedIn's $25/day practical minimum for Sponsored Content.
                     Recommendation: raise to $750 or switch to Text Ads ($10/day min).
Timing:              [Starts 2026-07-15, ends 2026-08-14]
Creative:            [PDF report — Document ad format]
Destination:         [eco.com/build]  (pixel: Insight Tag ✓ conversion event configured)
Tone:                [confident + technical]
Avoiding:            [em-dashes; "seamless/innovative/cutting-edge"; competitor names]

Defaults applied:
- audienceExpansionEnabled: false  (turn off LinkedIn's auto-widening)
- offsiteDeliveryEnabled: false    (LinkedIn Audience Network off — feed only)
- status: DRAFT                    (won't serve until you say "launch")
- optimizationTargetType: MAX_CLICK  (LinkedIn best-practice for cold traffic)
- creativeSelection: OPTIMIZED     (LinkedIn's default; picks best-performing variant)

Feasibility flags: 1 warning (see budget line above)

Reply with "launch" to activate, "adjust budget" / "adjust audience" / "adjust creative" to change something, or "cancel" to stop.
```

**Never activate on user silence or a "yes" that could be misread — require an explicit `launch` word.** After launch, echo the campaign ID + link back to the operator.

---

## Recommendation rules the skill applies

These are the "if X, recommend Y because Z" patterns pulled from each platform's best practices — the skill surfaces them proactively so the user knows why a default was chosen. Full per-platform rule set is in [routing-tables.md](./routing-tables.md). Highlights:

**Universal (all platforms):**
- Cold audience → auto-expansion flags OFF (Advantage+ Audience, LAN, Optimize Targeting, Smart Targeting, Search Partners)
- First-time on a platform → create as DRAFT/PAUSED, GET-verify state, then activate
- Regulated category detected → require the correct declaration BEFORE any create call, not after

**Per platform (compact — full detail in routing-tables.md):**
- **LinkedIn:** Sponsored Content minimum ~$25/day (CPCs $5–12); audience 50K–400K sweet spot; if `MAX_CONVERSION` and no Insight Tag conversion → refuse. Cold TOF for B2B → Document format outperforms single-image.
- **Meta:** Advantage+ Audience `=0` for first 2 weeks on cold audiences; interest count 1–3, not 10; if `daily_budget < 5× expected CPA` → warn (won't exit learning). If special_ad_categories set → detailed interest targeting is auto-disabled.
- **X:** Cold TOF → default to `REACH` (cheap CPM) not `WEBSITE_CLICKS`; conversion objectives need $50/day+ + a firing pixel. Custom audiences need ≥500 matched users (thin below 10K).
- **Google Search:** New account cold start → `TARGET_SPEND` (Maximize Clicks) or `MANUAL_CPC`, NOT `TARGET_CPA` (needs 15+ conversions/30d). Wrap keywords in `"quotes"` for phrase match unless specified. `target_google_search: true`, `target_search_network: false`, `target_content_network: false` for pure Search.
- **Reddit:** Community targeting (subreddits) outperforms interest categories on cold audiences. `expand_targeting: false` for first 2 weeks. Community <50K members → flag delivery risk but don't block (niche can convert well). Budget on ad group via `goal_type` + `goal_value` in micros.
- **TikTok:** `smart_audience_enabled` + `smart_interest_behavior_enabled` both OFF for cold audiences. Cold budget ≥20× target CPA/day at ad-group level. Creative not 9:16 → warn (feed is vertical-native). Monthly budget <$1,500 (~$50/day floor) → redirect to another platform.

---

## What routes where

The intake collects, then routes to the right sub-skill:

| Intake step | Handled by |
|---|---|
| 10 questions + platform-specific follow-ups + feasibility gates + confirmation summary | this file (intake/SKILL.md) |
| Routing tables lookup (enums, budget units, min budgets, targeting patterns, regulated declarations) | [routing-tables.md](./routing-tables.md) |
| Copy generation + anti-slop check | [../copy/SKILL.md](../copy/SKILL.md) |
| Actual campaign create/verify/delete on the platform | [../linkedin/](../linkedin/SKILL.md) · [../meta/](../meta/SKILL.md) · [../x/](../x/SKILL.md) · [../google/](../google/SKILL.md) · [../reddit/](../reddit/SKILL.md) · [../tiktok/](../tiktok/SKILL.md) |

Never skip the intake for a first-time user. Returning users who paste a complete brief can fast-path to the confirmation summary — but the feasibility gates still fire.
