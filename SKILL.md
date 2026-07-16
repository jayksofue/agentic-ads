---
name: agentic-ads
description: Launch paid ads on LinkedIn, Meta, X, Google, Reddit, or TikTok from a single guided walkthrough. Handles the right campaign objective, budget format, targeting, creative sizing, and regulated-category approvals per platform. Every campaign creates as a Draft or Paused — nothing goes live without an explicit "launch" from the user. Use when the user wants to create, review, or launch an ad campaign on any of these platforms.
---

# Agentic Ads — Claude Code Skill

**One prompt. Six ad platforms. Nothing goes live without your say-so.**

Turn Claude Code into a paid-media junior marketer that speaks LinkedIn, Meta, X, Google, Reddit, and TikTok. Answer 10 plain-English questions — the skill translates them into each platform's specifics: the right campaign objective, the right budget format and minimums, targeting that matches the audience you described, creative sized for the format, and any extra approvals you need if you're advertising a regulated category like finance, crypto, or health.

## Why this exists

- **Never launch by accident.** Every campaign creates as a Draft or Paused. Activation requires an explicit `launch` from you, after a full confirmation summary echoes back every default the skill applied.
- **Sanity checks catch waste before it costs money.** If your budget doesn't hit the platform's practical minimum, or you picked a conversion goal without a pixel installed, or you're in a regulated category without the right approval, or your creative doesn't fit the format — the skill blocks the launch with the actionable fix, not a silent warning.
- **Best-practice defaults, not bolt-ons.** The default settings that quietly widen audiences and drive up costs (LinkedIn LAN, Audience Expansion, Meta Advantage+ Audience, X Optimized Targeting, Google Search Partners + AI Max, Reddit Expand Targeting, TikTok Smart Targeting) are all **off by default**. Turn them on explicitly if you want them.
- **Sourced, not vibes.** Every setting the skill picks — objective names, budget formats, spend minimums, targeting toggles, regulated-category rules — is cross-checked against each platform's official docs (or well-known third-party integrations where the docs aren't public). See [routing tables](./intake/routing-tables.md). Unverifiable claims are flagged so you know what to test in the platform's sandbox before shipping.
- **One intake, six platforms.** Same 10 questions whether you're running a LinkedIn Document ad or a TikTok Spark Ad. Platform-specific extras (LinkedIn Job Function, Reddit subreddits, Google seed keywords, TikTok Spark auth) only asked when relevant.

## Supported platforms

| Platform | How Claude runs it | Campaigns created as |
|---|---|---|
| LinkedIn | Browser (Campaign Manager) | Draft |
| Meta | Marketing API via `meta-ads` CLI, or browser | Paused |
| X (Twitter) | Browser, or X Ads API v12 (approval-gated) | Draft |
| Google | Browser, or Google Ads API (approval-gated) | Paused |
| Reddit | Browser, or Reddit Ads API v3 (allow-list) | Paused |
| TikTok | Browser, or TikTok Marketing API v1.3 (approval-gated) | Paused (disabled) |

Per-platform verification status is tracked in each platform's SKILL.md — live-verified paths get ✅, partially verified with a documented gap get 🚧, documented but not exercised end-to-end here get 📝.

Browser paths always require the Claude for Chrome extension, logged into that platform.

## Quickstart

### Step 0 — Browser extension (required)

Every platform in this skill drives its Ads Manager UI in your real Chrome (as the primary path for LinkedIn/Reddit and as a browser fallback for Meta/X/Google/TikTok whose APIs are approval-gated). The Chrome extension is required across the board.

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Log into every ad platform you plan to use (LinkedIn Campaign Manager, business.facebook.com / ads.facebook.com, ads.x.com, ads.google.com, ads.reddit.com, ads.tiktok.com) in Chrome before starting

### Step 1 — Run the skill

1. Open Claude Code
2. Say: **"run agentic-ads"** (the skill folder registers itself by name)
3. Claude walks you through the **[10-question guided intake](./intake/SKILL.md)**: what you're promoting, goal, audience, geography, budget + timing, creative, landing destination, tone, and anything to avoid in the ad copy. Plus 1–2 platform-specific extras where they matter (LinkedIn: job function / seniority; Reddit: which subreddits; etc.).
4. Before creating anything, Claude runs **sanity checks**: budget vs platform minimum, pixel installed if you picked a conversion goal, correct approval if you're in a regulated category, creative that fits the format, and enough account access to actually create the campaign. If anything fails, it stops and tells you the specific fix.
5. Claude builds the campaign as a Draft (or Paused, depending on platform), echoes back a full confirmation summary — every default applied is called out — and only activates when you explicitly say `launch`.

## Skill map

- **[intake/SKILL.md](./intake/SKILL.md)** — the guided walkthrough (entry point). 10 core questions + platform-specific extras + the pre-launch sanity checks.
- **[intake/routing-tables.md](./intake/routing-tables.md)** — reference: which objective each platform uses for a given goal, budget formats and minimums, the auto-expansion setting names per platform, regulated-category rules, creative sizing per format. Every claim source-cited.
- **[LinkedIn](./linkedin/SKILL.md)** · **[Meta](./meta/SKILL.md)** · **[X](./x/SKILL.md)** · **[Google](./google/SKILL.md)** · **[Reddit](./reddit/SKILL.md)** · **[TikTok](./tiktok/SKILL.md)** — per-platform mechanics: how the browser / API path works, quirks to watch for, how to delete after a QA run.
- **[Copy generation + anti-slop](./copy/SKILL.md)** — writes ad copy in your voice, enforces platform character limits, catches slop.

## Dry run mode

Every platform supports a preview/validate step before any money is spent. See [dry-run.md](./dry-run.md).

## Prompt examples

```
Launch a document ad on LinkedIn targeting CFOs and treasurers in financial services.
$500 lifetime budget. July 1–31. LAN off. Audience expansion off. Leave as Draft.
```

```
Create a Meta awareness campaign for fintech decision-makers in the US.
$1,000 monthly budget. Use the creative at [URL]. Leave as Paused for review.
This is a financial-services ad — apply the right regulated-category declaration.
```

```
Deploy an X engagements campaign targeting crypto and DeFi audiences.
$300 total budget. Show me the summary before launching. Save as Draft.
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
