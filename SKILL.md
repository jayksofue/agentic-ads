# Agentic Ads — Claude Code Skill

Deploy LinkedIn, Meta, X, Google, and Reddit ads from a single prompt. No platform expertise required.

## What this skill does

You describe the campaign. Claude handles targeting, placements, budgets, creative attachment, and launch — across any supported platform.

## Supported platforms

| Platform | Method | Status |
|---|---|---|
| LinkedIn | Browser automation (Campaign Manager) | ✅ Production-ready (live launches at Eco) |
| Meta | `meta-ads` CLI (Marketing API) | ✅ Verified end-to-end 2026-07-04 (live create→delete) |
| X (Twitter) | Browser automation or X Ads API | ✅ Browser verified 2026-07-09 (live draft→delete); API QA pending (approval-gated) |
| Google | Browser automation or Google Ads API | ⚠️ Browser QA attempted 2026-07-09, blocked (reachable account suspended/canceled); API QA pending (approval-gated) |
| Reddit | Browser automation or Reddit Ads API | 🚧 New — see reddit/ |

> **Verification legend:** ✅ = live create→delete cycle run and confirmed. 📝 = commands/UI steps documented but not yet exercised end-to-end from this environment (blocked on API credentials that require platform approval). Browser-automation paths depend on the Claude for Chrome extension being logged into each platform.

## Quickstart

### Step 0 — Browser extension (required)

This skill drives ad platform UIs directly in your browser. The extension is required for LinkedIn and for browser fallback mode on X and Google.

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Open Chrome and log into the ad platform you want to use (LinkedIn Campaign Manager, Meta Business Manager, etc.)

### Step 1 — Run the skill

1. Open Claude Code
2. Say: **"Run the agentic-ads skill"**
3. Claude will ask:
   - Which platform?
   - What ad format? (awareness / engagement / conversion / document)
   - Who's the audience?
   - What's the budget and date range?
4. Claude builds, previews, and (with your confirmation) launches

## Platform setup guides

- [LinkedIn →](./linkedin/SKILL.md)
- [Meta →](./meta/SKILL.md)
- [X →](./x/SKILL.md)
- [Google →](./google/SKILL.md)
- [Reddit →](./reddit/SKILL.md)
- [Copy generation + anti-slop →](./copy/SKILL.md)

## Dry run mode

Every platform supports a preview/validate step before any money is spent. See [dry-run.md](./dry-run.md).

## Real-world results

Deployed at [Eco](https://eco.com) — 3 ad sets across US/CA/AU, Singapore, and UK launched in one session.
CTR: 3.9–4.6% vs. LinkedIn's 0.44% platform average (~10x).

## Prompt examples

```
Launch a document ad on LinkedIn targeting CFOs and treasurers in financial services.
$500 lifetime budget. July 1–31. LAN off. Audience expansion off.
```

```
Create a Meta awareness campaign for fintech decision-makers in the US.
$1,000 monthly budget. Use the creative at [URL]. Dry run first.
```

```
Deploy an X engagement campaign targeting crypto and DeFi audiences.
$300 budget. Show me the payload before launching.
```
