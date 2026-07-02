# Agentic Ads

Deploy LinkedIn, Meta, X, and Google ads with a single Claude Code prompt.

No platform expertise required. You define the audience and budget. Claude handles targeting, placements, creative attachment, and launch.

## Results

Deployed at [Eco](https://eco.com):
- 3 ad sets across US/CA/AU, Singapore, and UK — launched in one session
- CTR: 3.9–4.6% vs. LinkedIn's 0.44% platform average (~10x)

## Quickstart

1. Install [Claude Code](https://claude.ai/code)
2. Clone this repo and point Claude at it:
   ```
   claude --skill ./agentic-ads/SKILL.md
   ```
3. Tell Claude what you want to launch

## Platforms

| Platform | Method | Dry run |
|---|---|---|
| **LinkedIn** | Browser automation (Campaign Manager) | Draft mode |
| **Meta** | Marketing API via MCP | `validate_only: true` |
| **X** | X Ads API (requires approval) or browser | Payload preview |
| **Google** | Google Ads API (requires approval) or browser | `validate_only: true` |

## Platform guides

- [LinkedIn setup →](./linkedin/SKILL.md)
- [Meta setup →](./meta/SKILL.md)
- [X setup →](./x/SKILL.md)
- [Google setup →](./google/SKILL.md)
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
- **Meta**: Connects via the Meta MCP to call the Marketing API directly. No UI.
- **X**: Calls the X Ads API with OAuth 1.0a. Previews payload before any call.
- **Google**: Uses the Google Ads Python client library. Validates before submitting.

## License

MIT
