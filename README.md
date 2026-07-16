# Agentic Ads

**One prompt. Six ad platforms. Every launch is paused by default.**

Agentic Ads turns Claude Code into a paid-media junior marketer that speaks LinkedIn, Meta, X, Google, Reddit, and TikTok. Answer 10 plain-English questions once — Claude handles the objective enums, budget math, targeting spec, creative specs, cold-audience-expansion flags, and regulated-category declarations for whichever platform(s) fit your inputs.

## Why use this

**Never launch by accident.** Every campaign is created as Draft or Paused. Activating requires an explicit `launch` from you, after Claude echoes back a full confirmation summary — including every default it applied so nothing is hidden.

**Feasibility gates catch waste before it costs you.** Five hard checks run before any create call:
- Budget × duration hits the platform's practical minimum
- Conversion goal has a pixel firing with real event volume
- Regulated category has the correct declaration or certification
- Creative aspect ratio / duration / size matches the format
- Your account role can actually create campaigns

If any fail, Claude blocks the launch with the exact fix — not a silent warning that lets your money burn.

**Best practices baked in, not bolted on.** The auto-expansion flags that quietly widen your audience and inflate your CPM — LinkedIn Audience Network, Audience Expansion, Meta Advantage+ Audience, X Optimized Targeting, Google Search Partners + AI Max, Reddit Expand Targeting, TikTok Smart Targeting — are all **off by default**. Turn them on explicitly if you want them; you won't get them by accident.

**Sourced, not vibes.** Every objective enum, budget unit, minimum threshold, expansion-flag name, and regulated-category rule is cross-checked against primary docs or verified integrator schemas. The [routing tables](./intake/routing-tables.md) list every source URL. Where a claim couldn't be primary-sourced, it's called out at the top of that file so you know what to sandbox-verify before shipping.

**One intake, six platforms.** Same 10 questions whether you're running a LinkedIn Document ad or a TikTok Spark Ad. Platform-specific mechanics (LinkedIn Job Function + Seniority, Reddit subreddits, Google seed keywords, TikTok Spark authorization) get asked only when relevant.

**Copy that doesn't sound like an AI wrote it.** Anti-slop check catches "seamless / innovative / cutting-edge," passive voice, weak CTAs, and em-dashes-as-drama before the ad ships. Character limits enforced per platform (LinkedIn 200 hard / 70 to avoid truncation, Meta 40, Google RSA 3–15 headlines / 2–4 descriptions).

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

| Platform | Method | Draft/paused state |
|---|---|---|
| **LinkedIn** | Browser automation (Campaign Manager) | Draft |
| **Meta** | `meta-ads` PyPI CLI (Marketing API) | PAUSED |
| **X** | Browser automation or X Ads API v12 | Draft |
| **Google** | Browser automation or Google Ads API | Paused / Campaign Drafts |
| **Reddit** | Browser automation or Reddit Ads API v3 | `configured_status: PAUSED` |
| **TikTok** | Browser automation or TikTok Marketing API v1.3 | `operation_status: DISABLE` |

Full per-platform mechanics + gotchas live in the platform sub-skills below.

## Skill map

- **[Guided intake (start here) →](./intake/SKILL.md)** — the 10-question walkthrough + feasibility gates that runs before any campaign
- **[Routing tables →](./intake/routing-tables.md)** — verified goal→enum maps, budget units + minimums, regulated-category rules, creative specs, per-platform recommendation rules
- **[Copy generation + anti-slop →](./copy/SKILL.md)** — writes ad copy in your voice, enforces platform character limits, catches slop
- **[Dry run guide →](./dry-run.md)** — per-platform preview / no-spend paths
- Platform sub-skills: [LinkedIn](./linkedin/SKILL.md) · [Meta](./meta/SKILL.md) · [X](./x/SKILL.md) · [Google](./google/SKILL.md) · [Reddit](./reddit/SKILL.md) · [TikTok](./tiktok/SKILL.md)

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

1. **Prescriptive on safety, advisory on strategy.** The skill enforces what would fail or waste money (feasibility gates, correct enum names, correct pixel state, correct regulated-category declarations). It surfaces trade-offs where you actually have a choice — never picks the platform for you without knowing your inputs.
2. **Never launch on hidden defaults.** Every default is echoed in the confirmation summary before activation. Activation requires an explicit `launch` word.
3. **Honest about what's verified vs documented.** Live-verified paths get ✅. Partially verified with a documented gap get 🚧. Documented but not exercised end-to-end from this environment get 📝. The status table on the [root skill](./SKILL.md) reflects it per platform.

## License

MIT
