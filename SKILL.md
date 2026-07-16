---
name: agentic-ads
description: Deploy paid ads across LinkedIn, Meta, X, Google, Reddit, and TikTok from a single prompt. Handles targeting, placements, budgets, creative attachment, and launch via browser automation (Claude for Chrome) or platform APIs. Every platform defaults to paused/draft — nothing goes live without explicit confirmation. Use this when the user wants to create, review, or launch an ad campaign on any of these platforms.
---

# Agentic Ads — Claude Code Skill

Deploy LinkedIn, Meta, X, Google, Reddit, and TikTok ads from a single prompt. No platform expertise required.

## What this skill does

You describe the campaign. Claude handles targeting, placements, budgets, creative attachment, and launch — across any supported platform.

## Supported platforms

| Platform | Method | Status |
|---|---|---|
| LinkedIn | Browser automation (Campaign Manager) | ✅ Production-ready (live launches at Eco) |
| Meta | `meta-ads` PyPI CLI (Marketing API; blog-referenced, not formally Meta-attributed) | 🚧 Live create→delete 2026-07-04 via the third-party npm fallback (✅); PyPI CLI command surface verified against the binary — live re-run pending a fresh access token (📝) |
| X (Twitter) | Browser automation or X Ads API | ✅ Browser verified 2026-07-09 (live draft→delete); API QA pending (approval-gated) |
| Google | Browser automation or Google Ads API | 🚧 Create/draft verified 2026-07-09 (Eco 2025); saved-draft delete leg unverified — no UI delete once a draft is saved, so QA path is **Discard-on-exit**; publish gated by "Confirm it's you"; API QA pending (approval-gated) |
| Reddit | Browser automation or Reddit Ads API v3 | 📝 Documented, not live-verified. Paused-state field is **`configured_status`** (verified 2026-07-12 against Fivetran/dltHub/MCP schemas — audit's Critical finding resolved). API write scope requires Reddit ads-team allow-list. |
| TikTok | Browser automation or TikTok Marketing API v1.3 | 📝 Documented, not live-verified — see tiktok/ |

> **Verification legend:**
> - **✅** = live create→delete cycle run and confirmed on this environment
> - **🚧** = partially verified with a documented gap (see per-platform QA banner)
> - **📝** = commands/UI steps documented but not exercised end-to-end here (typically blocked on API credentials requiring platform approval, or on unverified field names — see per-platform QA banner)
> - **⚠️** = documented safety-critical caveat operators must resolve before shipping (currently: none — Reddit paused-field caveat was resolved 2026-07-12)
>
> Browser-automation paths always require the Claude for Chrome extension logged into that platform.

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

## Real-world results

Deployed at [Eco](https://eco.com) — **3 LinkedIn Campaigns** covering three geo clusters (US/CA/AU, Singapore, UK) launched in a single session.
CTR: 3.9–4.6% vs. LinkedIn's 0.44% platform average (~10×).

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
