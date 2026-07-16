---
name: agentic-ads
description: Deploy paid ads across LinkedIn, Meta, X, Google, Reddit, and TikTok from a single prompt. Handles targeting, placements, budgets, creative attachment, and launch via browser automation (Claude for Chrome) or platform APIs. Every platform defaults to paused/draft — nothing goes live without explicit confirmation. Use this when the user wants to create, review, or launch an ad campaign on any of these platforms.
---

# Agentic Ads — Claude Code Skill

**One prompt. Six ad platforms. Nothing goes live without your say-so.**

Turn Claude Code into a paid-media junior marketer that speaks LinkedIn, Meta, X, Google, Reddit, and TikTok. Answer 10 plain-English questions once — the skill handles objective enums, budget math, targeting spec, creative specs, cold-audience-expansion flags, and regulated-category declarations for whichever platform(s) fit your inputs.

## Why this exists

- **Never launch by accident.** Every campaign creates as Draft or Paused. Activation requires an explicit `launch` from the operator, after a full confirmation summary echoes back every default the skill applied.
- **Feasibility gates catch waste before it costs money.** Budget below the platform's practical minimum, conversion goal without a pixel firing, regulated category without the required declaration, creative that doesn't fit the format — all block the launch with the actionable fix, not a silent warning.
- **Best-practice defaults, not bolt-ons.** The auto-expansion flags that quietly widen audiences and inflate CPMs (LinkedIn LAN, Audience Expansion, Meta Advantage+ Audience, X Optimized Targeting, Google Search Partners + AI Max, Reddit Expand Targeting, TikTok Smart Targeting) are all **off by default**. Turn them on explicitly if you want them.
- **Sourced, not vibes.** Every enum, budget unit, minimum threshold, and regulated-category rule is cross-checked against primary platform docs or verified integrator schemas ([routing tables](./intake/routing-tables.md)). Unverifiable claims are flagged so you know what to sandbox-verify.
- **One intake, six platforms.** Same 10 questions whether you're running a LinkedIn Document ad or a TikTok Spark Ad. Platform-specific follow-ups (LinkedIn Job Function, Reddit subreddits, Google seed keywords, TikTok Spark auth) only asked when relevant.

## Supported platforms

| Platform | Method | Draft/paused state |
|---|---|---|
| LinkedIn | Browser automation (Campaign Manager) | Draft |
| Meta | `meta-ads` PyPI CLI (Marketing API) | PAUSED |
| X (Twitter) | Browser automation or X Ads API v12 | Draft |
| Google | Browser automation or Google Ads API | Paused / Campaign Drafts |
| Reddit | Browser automation or Reddit Ads API v3 | `configured_status: PAUSED` |
| TikTok | Browser automation or TikTok Marketing API v1.3 | `operation_status: DISABLE` |

Per-platform verification status is tracked per-file (see the QA banner in each platform's SKILL.md). Live-verified paths get ✅, partially verified with a documented gap get 🚧, documented-but-not-live-run get 📝.

Browser-automation paths always require the Claude for Chrome extension logged into that platform.

## Quickstart

### Step 0 — Browser extension (required)

Every platform in this skill drives its Ads Manager UI in your real Chrome (as the primary path for LinkedIn/Reddit and as a browser fallback for Meta/X/Google/TikTok whose APIs are approval-gated). The Chrome extension is required across the board.

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Log into every ad platform you plan to use (LinkedIn Campaign Manager, business.facebook.com / ads.facebook.com, ads.x.com, ads.google.com, ads.reddit.com, ads.tiktok.com) in Chrome before starting

### Step 1 — Run the skill

1. Open Claude Code
2. Say: **"run agentic-ads"** (the frontmatter registers the skill by name)
3. Claude walks you through the **[10-question guided intake](./intake/SKILL.md)**: what you're promoting, goal, audience, geo, budget + timing, creative, destination, tone, avoids. Plus 1–2 platform-specific questions (LinkedIn: job function/seniority; Reddit: subreddits; etc.).
4. Before any create call, Claude runs **feasibility gates**: platform budget minimums, pixel + volume for conversion goals, regulated-category approval status, creative-format match, ad-account access level. Fails loud with the fix, not a warning.
5. Claude builds the campaign PAUSED/DRAFT (never live), echoes back a full confirmation summary — including every default applied — and activates only on explicit `launch` from you.

## Skill map

- **[intake/SKILL.md](./intake/SKILL.md)** — the guided walkthrough (entry point). 10 core questions + platform-specific follow-ups + feasibility gates.
- **[intake/routing-tables.md](./intake/routing-tables.md)** — verified 2026-07-12 reference: goal→enum maps, budget units, minimums, cold-audience-expansion flag names, regulated-category rules, creative specs. Every claim source-cited.
- **[LinkedIn](./linkedin/SKILL.md)** · **[Meta](./meta/SKILL.md)** · **[X](./x/SKILL.md)** · **[Google](./google/SKILL.md)** · **[Reddit](./reddit/SKILL.md)** · **[TikTok](./tiktok/SKILL.md)** — per-platform mechanics: how the browser/API path works, gotchas, delete-after-QA.
- **[Copy generation + anti-slop](./copy/SKILL.md)** — copy generation + platform-specific character limits + slop check.

## Dry run mode

Every platform supports a preview/validate step before any money is spent. See [dry-run.md](./dry-run.md).

## Prompt examples

```
Launch a document ad on LinkedIn targeting CFOs and treasurers in financial services.
$500 lifetime budget. July 1–31. LAN off. Audience expansion off. Leave as Draft.
```

```
Create a Meta awareness campaign for fintech decision-makers in the US.
$1,000 monthly budget. Use the creative at [URL]. Create PAUSED (paused-review pattern; no --dry-run on the PyPI CLI).
Declare special-ad-category: FINANCIAL_PRODUCTS_SERVICES.
```

```
Deploy an X engagements campaign targeting crypto and DeFi audiences.
$300 total budget. Show me the payload before launching. Save as Draft.
```

## Ad review SLAs

Once you activate a campaign, most platforms hold it in review before serving. Rough windows:

| Platform | Typical review | Regulated (crypto/finance/politics/gambling) |
|---|---|---|
| LinkedIn | ≤24h | up to 48h |
| Meta | 15 min – few hours | 24h+, occasional discretionary rejections |
| X | ≤24h | 24–48h |
| Google | 1 business day | 3–5 business days, can require verification |
| Reddit | few hours – 24h | 24h+, some geos restricted for crypto |
| TikTok | 24h typical | 24–48h; strict on financial/investment claims |

Don't schedule launch windows that assume instant serving. Claude checks status after activation and reports back.
