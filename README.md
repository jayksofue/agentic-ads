# Agentic Ads

Deploy LinkedIn, Meta, X, Google, Reddit, and TikTok ads with a single Claude Code prompt.

No platform expertise required. You define the audience and budget. Claude handles targeting, placements, creative attachment, and launch.

## Results

Deployed at [Eco](https://eco.com):
- 3 ad sets across US/CA/AU, Singapore, and UK — launched in one session
- CTR: 3.9–4.6% vs. LinkedIn's 0.44% platform average (~10x)

## Quickstart

### Step 0 — Browser extension (required for every platform)

Every platform in this skill drives its Ads Manager UI in your real Chrome (as a fallback for the API-only platforms, and as the primary path for platforms without a usable API tier — LinkedIn, Reddit).

1. Install the **Claude for Chrome** extension from the Chrome Web Store
2. In Claude Code, enable it under Settings → MCP → Browser
3. Log into every ad platform you plan to use (LinkedIn Campaign Manager, ads.reddit.com, ads.x.com, ads.google.com, ads.tiktok.com, business.facebook.com) in Chrome before starting

### Step 1 — Install as a Claude Code skill

1. Install [Claude Code](https://claude.ai/code)
2. Clone this repo and install it as a skill by symlinking (or copying) into your Claude skills directory:
   ```bash
   git clone https://github.com/jayksofue/agentic-ads.git
   mkdir -p ~/.claude/skills
   ln -s "$PWD/agentic-ads" ~/.claude/skills/agentic-ads
   ```
   (Or drop the folder in `.claude/skills/` inside a specific project to scope it there.) The skill's YAML frontmatter registers it as `agentic-ads`.
3. In Claude Code, say **"run agentic-ads"** (or reference the skill by name). Claude will ask which platform + what you want to launch.

## Platforms

| Platform | Method | Dry run |
|---|---|---|
| **LinkedIn** | Browser automation (Campaign Manager) | Draft status |
| **Meta** | `meta-ads` PyPI CLI (blog-referenced, not formally Meta-attributed) | Paused campaign, then delete (no dry-run flag; npm fallback has one) |
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

- **LinkedIn**: Drives Campaign Manager via browser using the `nativeSetter` pattern for React inputs. Handles LAN, Audience Expansion, date pickers, and format selection automatically. Creates as Draft.
- **Meta**: Calls the Marketing API directly through the `meta-ads` PyPI CLI (Python, `pip install meta-ads`, binary `meta`). This is the CLI referenced in Meta's April 2026 "Introducing Ads CLI" developer blog post — PyPI author metadata is anonymous, so the skill treats it as blog-referenced rather than formally attributed. Live create→delete verified 2026-07-04 (via the third-party npm `meta-ads` fallback); PyPI CLI's commands + flag surface verified against the binary, live re-run pending a fresh access token.
- **X**: Drives Ads Manager via browser. Handles objective-specific ad formats (website card for traffic, standard post for reach/engagements/video). Turns off Optimize Targeting by default. Save-draft-from-Review path live-verified 2026-07-09.
- **Google**: Drives Google Ads via browser. Handles campaign type selection, Video subtypes, turns off Display Network and AI Max (Final URL expansion + Text customization) by default. Create/draft flow live-verified 2026-07-09 on Eco 2025 — note saved drafts have no UI delete, so QA path is Discard-on-exit.
- **Reddit**: Drives ads.reddit.com via browser (primary) or the Reddit Ads API v3 (allow-list required). Subreddit/community targeting is the platform's main advantage. API paused-field name unverified against Reddit's private v3 spec — see reddit/ safety banner.
- **TikTok**: Drives ads.tiktok.com via browser or calls the TikTok Marketing API v1.3 (developer-app review required). Video-first; enforces TikTok-only placements (not Pangle) and disables Automatic Targeting by default.

## License

MIT
