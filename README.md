# Agentic Ads

**One prompt. Six ad platforms. Nothing goes live without your say-so.**

Agentic Ads turns Claude Code into a paid-media junior marketer that speaks LinkedIn, Meta, X, Google, Reddit, and TikTok. Answer 10 plain-English questions — Claude translates them into each platform's specifics: the right campaign objective, the right budget format and minimums, targeting that matches the audience you described, creative sized for the format, and any extra approvals you need if you're advertising a regulated category like finance, crypto, or health.

## Why use this

**Never launch by accident.** Every campaign is created as a Draft or Paused. Activating requires an explicit `launch` from you, after Claude echoes back a full confirmation summary — including every default it applied so nothing is hidden.

**Sanity checks catch mistakes before they cost you.** Five hard checks run before any campaign gets created:
- Your budget × duration meets the platform's practical minimum
- If you picked a conversion goal, you have a pixel installed and it's actually seeing conversions
- If you're in a regulated category (crypto, finance, health, etc.), you have the right approval or declaration in place
- Your creative aspect ratio, duration, and file size fit the ad format
- Your account role can actually create campaigns (not just view them)

If any fail, Claude blocks the launch with the exact fix — not a silent warning that lets your money burn.

**Best practices baked in, not bolted on.** The default settings that quietly widen your audience and drive up your costs — LinkedIn Audience Network, Audience Expansion, Meta Advantage+ Audience, X Optimized Targeting, Google Search Partners + AI Max, Reddit Expand Targeting, TikTok Smart Targeting — are **all off by default**. Turn them on explicitly if you want them; you won't get them by accident.

**Sourced, not vibes.** Every setting the skill picks — objective names, budget formats, spend minimums, targeting toggles, regulated-category rules — is cross-checked against each platform's official docs (or well-known third-party integrations where the platform's docs aren't public). The [routing tables](./intake/routing-tables.md) list every source URL. Where a claim couldn't be primary-sourced, it's flagged so you know what to test in the platform's sandbox before shipping.

**One intake, six platforms.** Same 10 questions whether you're running a LinkedIn Document ad or a TikTok Spark Ad. Platform-specific extras (LinkedIn Job Function + Seniority, Reddit subreddits, Google seed keywords, TikTok Spark authorization) only get asked when relevant.

**Copy that doesn't sound like an AI wrote it.** An anti-slop check catches "seamless / innovative / cutting-edge," passive voice, weak CTAs, and em-dashes-as-drama before the ad ships. Character limits enforced per platform (LinkedIn 200 max, or 70 to avoid the "…see more" cutoff; Meta 40; Google Search 3–15 headlines × 30 chars + 2–4 descriptions × 90 chars).

## Quickstart

### 1. Install the browser extension

The skill drives each platform's Ads Manager UI in your real Chrome — required across the board (some platforms have API paths too, but browser is always the fallback):

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Log into every ad platform you plan to use (LinkedIn Campaign Manager, business.facebook.com, ads.x.com, ads.google.com, ads.reddit.com, ads.tiktok.com) in Chrome before starting

### 2. Install as a Claude Code skill

```bash
git clone https://github.com/jayksofue/agentic-ads.git
mkdir -p ~/.claude/skills
ln -s "$PWD/agentic-ads" ~/.claude/skills/agentic-ads
```

Or drop the folder in `.claude/skills/` inside a specific project to scope it there. The skill's YAML frontmatter registers it as `agentic-ads`.

### 3. Run it

In [Claude Code](https://claude.ai/code):

```
run agentic-ads
```

Claude walks you through the 10-question intake, runs the feasibility gates, echoes back everything it's about to do, and waits for your `launch`.

## Platforms

| Platform | How Claude runs it | Campaigns created as |
|---|---|---|
| **LinkedIn** | Browser (Campaign Manager) | Draft |
| **Meta** | Marketing API via `meta-ads` CLI, or browser | Paused |
| **X** | Browser, or X Ads API v12 (approval-gated) | Draft |
| **Google** | Browser, or Google Ads API (approval-gated) | Paused |
| **Reddit** | Browser, or Reddit Ads API v3 (allow-list) | Paused |
| **TikTok** | Browser, or TikTok Marketing API v1.3 (approval-gated) | Paused (disabled) |

Full per-platform mechanics, quirks, and cleanup steps live in the platform guides linked below.

## Skill map

- **[Guided intake (start here) →](./intake/SKILL.md)** — the 10-question walkthrough + sanity checks that runs before any campaign
- **[Routing tables →](./intake/routing-tables.md)** — which objective each platform uses for a given goal, budget formats and minimums, regulated-category rules, creative sizing per format, and platform-specific recommendations
- **[Copy generation + anti-slop →](./copy/SKILL.md)** — writes ad copy in your voice, enforces platform character limits, catches slop
- **[Dry run guide →](./dry-run.md)** — how to preview a campaign on each platform without spending
- Per-platform guides: [LinkedIn](./linkedin/SKILL.md) · [Meta](./meta/SKILL.md) · [X](./x/SKILL.md) · [Google](./google/SKILL.md) · [Reddit](./reddit/SKILL.md) · [TikTok](./tiktok/SKILL.md)

## Example prompts

Once you're set up, any of these works. Claude asks anything it needs and confirms before creating:

```
Run agentic-ads. I want to launch a LinkedIn Document ad for our CFO
audience in financial services. $500 lifetime, US/UK/Singapore.
```

```
Run agentic-ads. Cold prospecting on Meta for a B2B fintech product.
US developers, $1000/mo. I have a 1:1 image and a landing page.
```

```
Run agentic-ads. Reddit — target r/webdev, r/programming, r/devops.
$50/day for 2 weeks. Promote our existing post [url]. Traffic goal.
```

```
Run agentic-ads. Google Search for "stablecoin payments API" and related
B2B keywords. Cold start, $30/day, no conversion history yet.
```

## Design principles

1. **Prescriptive on safety, advisory on strategy.** The skill enforces things that would fail or waste money (budget minimums, correct objective for your goal, pixel installed for conversion campaigns, correct approval for regulated categories, creative that fits the format). It presents options where you actually have a choice — it never picks the platform for you without knowing your inputs.
2. **Never launch on hidden defaults.** Every default the skill applied is echoed in the confirmation summary before activation. Activation requires an explicit `launch` word from you.
3. **Honest about what's verified vs documented.** Live-verified paths get ✅. Partially verified with a documented gap get 🚧. Documented but not exercised end-to-end from this environment get 📝. Each platform's file tracks its own status.

## License

MIT
