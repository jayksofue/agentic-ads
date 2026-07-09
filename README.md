# Agentic Ads

Deploy LinkedIn, Meta, X, Google, Reddit, and TikTok ads with a single Claude Code prompt.

No platform expertise required. You define the audience and budget. Claude handles targeting, placements, creative attachment, and launch.

## Results

Deployed at [Eco](https://eco.com):
- 3 ad sets across US/CA/AU, Singapore, and UK — launched in one session
- CTR: 3.9–4.6% vs. LinkedIn's 0.44% platform average (~10x)

## Quickstart

### Step 0 — Browser extension (required)

This skill drives ad platform UIs directly in your browser. Install it first.

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Open Chrome and log into the ad platform you want to use

### Step 1 — Run the skill

1. Install [Claude Code](https://claude.ai/code)
2. Clone this repo and load the skill:
   ```
   claude --skill ./agentic-ads/SKILL.md
   ```
3. Tell Claude what you want to launch

## Platforms

| Platform | Method | Dry run |
|---|---|---|
| **LinkedIn** | Browser automation (Campaign Manager) | Draft status |
| **Meta** | `meta-ads` CLI (Marketing API) — verified end-to-end | `--dry-run` + paused campaign |
| **X** | Browser automation or X Ads API (requires approval) | Paused campaign |
| **Google** | Browser automation or Google Ads API (requires approval) | Paused campaign / Campaign Drafts |
| **Reddit** | Browser automation or Reddit Ads API | Paused campaign |
| **TikTok** | Browser automation or TikTok Marketing API | Disabled campaign / sandbox |

## Platform guides

- [LinkedIn setup →](./linkedin/SKILL.md)
- [Meta setup →](./meta/SKILL.md)
- [X setup →](./x/SKILL.md)
- [Google setup →](./google/SKILL.md)
- [Reddit setup →](./reddit/SKILL.md)
- [TikTok setup →](./tiktok/SKILL.md)
- [Dry run guide →](./dry-run.md)

## Example prompts

```
Launch a LinkedIn document ad targeting CFOs in financial services.
$500 lifetime budget. July 1–31. LAN off. Audience expansion off.
```

```
Create a Meta engagement campaign for fintech decision-makers in the US.
$1,000 monthly budget. Dry run first.
```

```
Deploy an X promoted post campaign targeting crypto and DeFi audiences.
$300 budget. Show me the payload before launching.
```

```
Create a Google Search campaign for B2B payments keywords.
$50/day. Target CPA: $15. Validate only — don't submit yet.
```

## How it works

Claude uses platform-specific methods per ad network:

- **LinkedIn**: Drives Campaign Manager via browser using the `nativeSetter` pattern for React inputs. Handles LAN, Audience Expansion, date pickers, and format selection automatically.
- **Meta**: Calls the Marketing API directly through the `meta-ads` CLI. No UI. Verified end-to-end (create→delete) on 2026-07-04.
- **X**: Drives Ads Manager via browser. Handles objective-specific ad formats (website card for traffic, standard post for reach/engagements/video). Turns off Optimize Targeting by default.
- **Google**: Drives Google Ads via browser. Handles campaign type selection, Video subtypes, turns off Display Network and AI Max (Final URL expansion + Text customization) by default.

## License

MIT
