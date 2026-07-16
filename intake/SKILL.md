---
name: agentic-ads-intake
description: Guided walkthrough for launching an ad campaign — asks 10 plain-English questions, applies each platform's best practices, and blocks anything that would fail platform policy or waste your money before you launch. Routes to the right platform guide (LinkedIn / Meta / X / Google / Reddit / TikTok) based on your answers. This is the entry point for agentic-ads.
---

# Intake — Guided Ad-Campaign Walkthrough

Ask 10 questions. Apply the platform-specific settings in [routing-tables.md](./routing-tables.md). Load whichever platform's guide fits the answers. Block the launch if any of the pre-launch checks fail.

**How the skill decides what to do vs. what to ask:** the skill enforces the things that would fail policy or waste your money — no debate. It surfaces options where you actually have a choice. It never picks the platform for you without knowing your inputs, but it will rule out platforms your budget/audience/creative/regulated-category doesn't fit.

---

## The 10-question walkthrough

Ask these in order. Each question has a **why-it-matters** line, an **example**, and a **default** you can accept with one word ("use default," "same as last," "your call"). If you paste a full brief up front (a longer message covering several of the questions), the skill skips ahead to the confirmation summary and only asks about anything still missing.

### 1. What are you promoting? (brand + product in one line)

> **Why it matters:** anchors the ad copy the skill writes, and tells Claude whether your ad falls into a regulated category (finance, crypto, health, employment, housing, gambling, politics, alcohol).
> **Examples:** "our SaaS invoicing tool for freelancers" · "Nike Pegasus 41 running shoes" · "our webinar on incident response for SREs" · "our B2B stablecoin payments infrastructure"

From this answer, Claude figures out whether the ad is in a regulated category (`financial_services`, `crypto`, `health`, `employment`, `housing`, `credit`, `gambling`, `politics`, `alcohol`, or `none`). If it matters — because that category needs extra approvals on certain platforms — Claude confirms with you before moving on. For example: *"This looks like a financial-services ad — is that right? It'll change which platforms can run it and what approvals you need."*

### 2. What's the goal?

> **Why it matters:** the same goal has a different name on every platform, so Claude picks the right name for whichever platform(s) you end up on.
> **Options:** `awareness` · `traffic` · `engagement` · `video_views` · `leads` · `conversions` · `app_installs` · `sales` · `followers` (X only)
> **Default:** `traffic` if you've got a landing page; `awareness` if you don't.

If you pick `conversions`, `sales`, or `leads`, Claude follows up: **"Do you have a pixel installed on the landing page, and has it recorded 30+ conversions in the last month?"** If not, Claude warns you and offers to fall back to `traffic` — Meta, Google, TikTok, and Reddit all need real conversion volume before their algorithms can optimize for conversions. Without it, a "conversion" campaign underdelivers because the platform has nothing to learn from.

### 3. Ideal customer in one paragraph

> **Why it matters:** feeds the targeting on every platform and the copy the skill writes.
> **Examples:** "US CTOs and heads of engineering at fintech startups, Series A–C, actively integrating payments infrastructure" · "US women 25–34 who follow running influencers and shop Nike/Lululemon"
> **Default:** ask — this is the highest-leverage input. The more specific, the better the targeting.

From the paragraph Claude pulls: demographics (age, gender, geography, language), professional attributes (role, industry, seniority, company size — only for B2B), interests/behaviors, and purchase intent. Each dimension gets mapped to the right targeting option on whichever platform runs the campaign (details in [routing-tables.md](./routing-tables.md)).

### 4. Where do they live? (countries + optional cities)

> **Why it matters:** sets the geographic targeting, and for regulated categories, decides which platforms even allow the ad in that geo.
> **Examples:** "US, UK, Singapore" · "US only" · "US + Canada + EU"
> **Default:** infer from Q3 (if the audience paragraph mentioned countries, use those).

For regulated categories, Claude checks whether the platform allows ads for that category in the geos you named. For example: crypto ads on TikTok are prohibited in the US but allowed in some Latin American markets with local licensing. If the target geo isn't on the platform's allowed list for the category, that platform gets ruled out for this combination.

### 5. Total budget + shape

> **Why it matters:** picks the budget structure and checks whether you're above each platform's practical minimum.
> **Options:** `$X lifetime over N days` · `$X/day always-on` · `$X total for a fixed campaign`
> **Examples:** "$500 lifetime over 30 days" · "$50/day for a month" · "$1,500 monthly recurring"
> **Default:** ask — no reasonable default here.

Claude immediately runs the **budget sanity check** (see the checks section below). Lifetime budget gets converted to a daily figure (lifetime ÷ days), then compared to each platform's practical minimum. If your daily spend falls below the platform's minimum, Claude rules that platform out and explains why.

### 6. Timing

> **Why it matters:** sets start/end dates, and warns you if platform review times won't leave you enough runway.
> **Options:** `ASAP` · `starts [date]` · `runs [start] to [end]` · `always-on until paused`
> **Default:** `ASAP` (start tomorrow so the ad has time for review).

Claude cross-checks against each platform's typical ad-review time (in the root [SKILL.md](../SKILL.md)). If you have a hard deadline that doesn't leave enough time — especially for regulated categories, where review can take 3–5 business days on Google or weeks on Meta/LinkedIn/Reddit/TikTok — Claude flags it and suggests: (a) start now, (b) switch to a `traffic` goal on a non-regulated platform where the review is faster, or (c) accept the delay.

### 7. What creative do you have?

> **Why it matters:** determines the ad format, which determines which platforms fit.
> **Options:** `video 9:16` (vertical — TikTok-native, works on Reels/Shorts) · `video 16:9` (horizontal — YouTube, LinkedIn) · `video 1:1` (square — Meta Feed, X) · `single image` · `carousel` · `document/PDF` (LinkedIn only) · `existing post to promote` · `nothing yet — help me plan`
> **Default:** ask — the creative decides which platforms are even in the running.

If you have no creative and no time to make one, Claude offers three paths: (a) copy-only formats (X promoted post, Reddit text post, Google Search text ads); (b) Claude writes the copy and you supply one image (works for Meta and LinkedIn single-image ads); (c) delay the campaign until creative is ready. Copy work routes through [copy/SKILL.md](../copy/SKILL.md).

Claude also runs a **creative-format sanity check**: if you're trying to run a Document ad on any platform other than LinkedIn (unavailable), or a 9:16 vertical video on LinkedIn (works but underperforms because LinkedIn feed is horizontal), the skill warns or blocks — depending on how bad the mismatch is.

### 8. Where should clicks go? (landing destination)

> **Why it matters:** sets the destination URL and decides whether conversion tracking is possible.
> **Options:** `landing page URL` · `app store link` · `existing lead form` · `phone number` · `nothing — awareness only`
> **Examples:** "yourbrand.com/pricing" · "apps.apple.com/…"
> **Default:** ask if the goal is `traffic` / `conversions` / `leads` / `app_installs`; skip if the goal is `awareness` / `video_views`.

If the goal is `conversions` but there's no pixel on the destination, Claude stops and says so plainly: *"We can't optimize for conversions on a page with no pixel. Your options: (a) install the platform's pixel first, (b) switch this campaign to Traffic and re-run as Conversions once you have data, (c) skip."* No silent fallback — it's your money, so you decide.

### 9. Tone / brand voice

> **Why it matters:** shapes the ad copy Claude writes.
> **Options:** `confident + technical` · `friendly + accessible` · `authoritative + formal` · `playful + irreverent` · `provide examples of past copy` · `same as our website`
> **Default:** `confident + technical` for B2B; `friendly + accessible` for consumer; if unsure, Claude infers from Q1.

Tone plus the "avoid" list from Q10 gets passed to [copy/SKILL.md](../copy/SKILL.md), which handles the actual writing.

### 10. Anything to avoid in the ad?

> **Why it matters:** brand-safety guardrails and slop rules.
> **Examples:** "no em-dashes, no 'seamless / innovative / cutting-edge,' no 'game-changer'" · "don't mention specific competitors" · "don't imply guaranteed returns"
> **Default:** apply the standard anti-slop checklist from [copy/SKILL.md](../copy/SKILL.md).

For regulated categories (crypto, finance, health), some phrases are prohibited by platform policy regardless of what you say. Claude enforces those either way — e.g. no "guaranteed returns" on crypto ads, on any platform.

---

## Platform-specific follow-up questions (only asked when relevant)

After the 10 core questions, Claude asks 1–2 platform-specific extras — but only if the platform you picked actually needs them:

| If you picked | Also asks |
|---|---|
| **LinkedIn** | Job function + seniority + company size band (LinkedIn's B2B targeting is unique — no other platform has this dimension) |
| **Meta** | Custom-audience or lookalike-audience sources — only if there's pixel data available to build from |
| **X** | Follower-lookalike @handles + keyword targeting (X's interest model works differently from the others) |
| **Google Search** | Seed keywords (2–5) and match-type preference (phrase vs. exact) |
| **Google Video / Demand Gen** | YouTube video URL (the asset has to exist on your channel) |
| **Google Performance Max** | Source landing page URL (Google infers the rest from it) |
| **Reddit** | Which subreddits to target (this is Reddit's main lever — Claude always asks, never auto-picks) |
| **TikTok** | Spark Ad (promoting an existing TikTok post) or fresh upload? If Spark, the post URL + Creator Marketplace authorization code |

If you're running on multiple platforms in one session (see below), Claude asks each platform's follow-ups separately.

---

## Multi-platform mode

**Supported:** launching the same brief across several platforms as **separate, parallel campaigns**. Same audience intent + same creative, but each platform gets its own campaign, its own budget, and its own reporting.

**Not supported (yet):** true cross-platform campaigns that share one budget and roll up into one report. Every ad platform's account is independent, and this skill doesn't try to allocate spend across them automatically. That's a later-version feature.

If you say "run this on LinkedIn and Meta," Claude treats it as two separate campaign builds. If you say "run on all platforms that fit," Claude tells you which ones pass the pre-launch checks and lets you pick.

---

## Pre-launch sanity checks

Run **before creating any campaign.** If any check fails, Claude blocks the launch and tells you the exact fix — never a silent warning that leaves you to figure it out.

### A. Budget × duration meets the platform minimum

Convert lifetime to daily (lifetime ÷ days). Compare against each platform's practical minimum:

| Platform | Practical daily minimum |
|---|---|
| Meta | ~$20/day (needs about 50 conversion events per week to exit the learning phase) |
| LinkedIn | $25/day for Sponsored Content (Text Ads work from $10/day) |
| X | ~$5–10/day for engagement/reach objectives; **~$50/day for conversion objectives** |
| Google Search | About 10× the expected cost-per-click (~$10–50/day depending on keyword competition) |
| Reddit | $50–100/day for the algorithm to actually learn (technical minimum is $5/day but the campaign starves at that level) |
| TikTok | $50/day at the campaign level, $20/day at the ad-group level — the platform hard-blocks smaller budgets |

If your budget falls below → **block with the exact fix**:
> "$100 lifetime × 30 days = $3.30/day. TikTok requires ~$50/day and Meta needs ~$20/day. Two options: raise the budget to ~$600 lifetime over 30 days, or switch to a Traffic goal on X or Reddit (both have lower minimums)."

### B. Conversion goal has a pixel + real volume

If the goal is `conversions`, `sales`, or `leads` on Meta:

1. Is a pixel installed on the destination? (you tell Claude)
2. Has that pixel recorded 30+ conversion events in the last 30 days?

If either answer is no → **block**:
> "Optimizing for conversions needs 30+ recent conversion events on file so the platform's algorithm has something to learn from. Your options: (a) install the pixel and drive 30+ events via a Traffic campaign first, then re-launch as Conversions, (b) run this campaign as Traffic right now and switch to Conversions later, (c) skip."

### C. Regulated category has the right approval

For each platform × regulated-category combination in [routing-tables.md](./routing-tables.md):

| Category | Meta | LinkedIn | X | Google | Reddit | TikTok |
|---|---|---|---|---|---|---|
| Cryptocurrency (any) | Authorizations & Verifications approval in Business Suite (days–weeks) | Sales-rep whitelist (days–weeks) | Category certification (days) | Cryptocurrencies certification (3–5 business days) | Sales rep required (multi-week) | **US: prohibited.** Latin American geos only, and only with a Sales Rep |
| Financial services | India-specific category setting, or Authorizations & Verifications | Sales-rep whitelist | Financial services certification | Financial products certification | Sales rep required | Sales Rep + Verified Business Account |
| Employment / Housing / Credit (US anti-discrimination) | Set the "special ad categories" flag for the right category | Automatically flagged based on account | Similar restrictions | Standard flow | Automatic | TikTok's `special_industries` setting |

If you haven't done the required approval on the platform you picked → **block with the fix**:
> "Meta requires Authorizations & Verifications approval for crypto ads. Your options: (a) start the approval now (days–weeks to complete), (b) route to X instead — the Cryptocurrency certification is fastest, usually a few days, (c) route to Google — Crypto certification takes 3–5 business days, (d) build a Reddit draft in the browser now, but you'll need a Reddit sales rep before you can publish it."

### D. Creative fits the ad format

Compare your creative's dimensions to each candidate platform's format requirements: aspect ratio, resolution, duration, file size. If the creative doesn't fit → warn if it's a soft mismatch (a horizontal video works on Meta Feed but underperforms a square one), block if it's a hard mismatch (a 100 MB image is over Meta's 30 MB size cap).

### E. You have the right account access

Before building, Claude asks: *"Are you an Admin or Advertiser on the [platform] ad account — not just a Viewer?"* Viewer-level accounts can't create campaigns. If you're not sure, Claude points you to the account settings page on that platform.

---

## The confirmation summary

Before creating anything, Claude echoes back the full setup. Every default the skill applied gets listed explicitly — nothing hidden.

Template:

```
Ready to build. Confirming your setup:

Platform:            [LinkedIn]
Objective:           [Website Visit / traffic]  ← from your goal "traffic"
Audience:            [US CTOs, engineering leads at fintech Series A–C]
                     LinkedIn targeting: Function = Engineering + IT · Seniority = Director + VP + CXO
                     Industry = Financial Services + Software · Company size = 51–1,000
Estimated reach:     [~180K members]  ← within LinkedIn's 50K–400K sweet spot ✓
Geo:                 [US, UK, Singapore]
Budget:              [$500 lifetime over 30 days = ~$16.70/day]
                     ⚠ Below LinkedIn's $25/day practical minimum for Sponsored Content.
                     Recommendation: raise to $750, or switch to Text Ads ($10/day min).
Timing:              [Starts 2026-07-15, ends 2026-08-14]
Creative:            [PDF report — Document ad format]
Destination:         [yourbrand.com/build]  (LinkedIn Insight Tag installed, conversion event configured ✓)
Tone:                [confident + technical]
Avoiding:            [em-dashes; "seamless / innovative / cutting-edge"; competitor names]

Defaults the skill applied:
- Audience Expansion: OFF (turn off LinkedIn's auto-widening)
- LinkedIn Audience Network: OFF (LinkedIn feed only)
- Status: DRAFT (won't serve until you say "launch")
- Bidding: Maximum Delivery — optimize for clicks (best practice for cold traffic on LinkedIn)
- Creative rotation: OPTIMIZED (LinkedIn picks the best-performing variant automatically)

Sanity checks: 1 warning (see budget line above), everything else passed.

Reply with "launch" to activate. To change something, say "adjust budget" / "adjust audience" / "adjust creative." To stop, say "cancel."
```

**Never activate on silence or a "yes" that could be misread — Claude requires the literal word `launch`.** After launch, Claude echoes back the campaign ID and a link to it in Ads Manager.

---

## What Claude recommends by default (and why)

These are the "if X, recommend Y because Z" patterns Claude applies proactively — surfacing them so you know why a default was picked. The full per-platform list lives in [routing-tables.md](./routing-tables.md); the highlights:

**Every platform:**
- **Cold audience** → the platform's audience-expansion setting gets turned OFF (Meta Advantage+ Audience, LinkedIn LAN + Audience Expansion, X Optimized Targeting, Google Search Partners + AI Max, Reddit Expand Targeting, TikTok Smart Targeting). Turn them on later if you want to test them; you shouldn't get them by accident.
- **First-time on a platform** → create as Draft or Paused, verify it saved that way, only then activate.
- **Regulated category detected** → require the right approval BEFORE creating the campaign — not "we'll deal with it in review."

**Per platform (highlights — full list in [routing-tables.md](./routing-tables.md)):**
- **LinkedIn:** Sponsored Content minimum $25/day (cost-per-click runs $5–12, so smaller budgets can't generate enough signal). Sweet-spot audience size 50K–400K. If you pick Conversions but haven't set up an Insight Tag conversion event, Claude refuses to activate. For cold B2B traffic, Document ads outperform single-image.
- **Meta:** For cold audiences, turn Advantage+ Audience off for the first two weeks and use 1–3 interests, not 10. If daily budget is less than 5× your expected cost-per-acquisition, warn — the campaign won't exit learning phase.
- **X:** For cold prospecting, default to Reach (cheap CPM) not Website Clicks. Conversion objectives need $50/day+ and a firing pixel. Custom Audience uploads need at least 500 matched users — below 10K, delivery gets thin.
- **Google Search:** New account with no conversion history → use Maximize Clicks or Manual CPC, not Target CPA (which needs 15+ conversions in 30 days to work). Wrap keywords in `"quotes"` for phrase match unless you specify otherwise. Turn OFF Search Partners and Display Network for pure Search campaigns.
- **Reddit:** Subreddit targeting outperforms interest-category targeting on cold audiences. Leave Expand Targeting off for the first two weeks. A subreddit under 50K members flags for possible delivery risk but doesn't block — niche subs can convert well.
- **TikTok:** Both Smart Targeting settings (Smart Audience and Smart Interest & Behavior) off for cold audiences. Cold audience budget should be ≥20× your target cost-per-acquisition, at the ad-group level. If your creative isn't 9:16 vertical, warn — TikTok feed is vertical-native. If your monthly budget is under $1,500 (the ~$50/day floor), redirect to another platform.

---

## What routes where

The intake collects everything, then routes to the right file:

| Step | Handled by |
|---|---|
| The 10 questions, platform-specific follow-ups, sanity checks, confirmation summary | This file (intake/SKILL.md) |
| Which objective each platform uses for a given goal, budget formats, minimum budgets, targeting patterns, regulated-category rules, creative sizing | [routing-tables.md](./routing-tables.md) |
| Copy generation + anti-slop check | [../copy/SKILL.md](../copy/SKILL.md) |
| Actual campaign create → verify → delete on the platform | [../linkedin/](../linkedin/SKILL.md) · [../meta/](../meta/SKILL.md) · [../x/](../x/SKILL.md) · [../google/](../google/SKILL.md) · [../reddit/](../reddit/SKILL.md) · [../tiktok/](../tiktok/SKILL.md) |

Never skip the intake for a first-time user. Returning users who paste a full brief can fast-path to the confirmation summary — but the sanity checks still fire before launch.
