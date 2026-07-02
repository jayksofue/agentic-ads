# LinkedIn Ads — Skill

Claude drives LinkedIn Campaign Manager directly via browser automation. No API key required.

## Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, then enable under Claude Code Settings → MCP → Browser. This is what lets Claude drive the Campaign Manager UI.
2. **Claude Code** — claude.ai/code
3. **Logged into LinkedIn Campaign Manager in Chrome** — navigate to linkedin.com/campaignmanager before starting
4. **Account ID** — find it in the Campaign Manager URL: `/accounts/{ID}/`

## How it works

Claude navigates Campaign Manager, fills in all fields using the `nativeSetter` pattern (required for React-controlled inputs), and launches the ad set. You confirm before anything goes live.

## Campaign setup prompt

```
Launch a LinkedIn sponsored content campaign:
- Account ID: [your account ID]
- Campaign group: [name or ID]
- Ad format: Document / Single image / Video
- Audience: [job titles, functions, industries, interests]
- Locations: [countries or regions]
- Budget: $[amount] lifetime, [start date]–[end date]
- Creative: [post URL or creative ID]
- LAN: off
- Audience expansion: off
```

## Key gotchas Claude handles automatically

| Issue | Fix |
|---|---|
| React inputs ignore `.value =` | Uses `nativeSetter` via `Object.getOwnPropertyDescriptor` |
| LinkedIn Audience Network on by default | Unchecks index 3 checkbox after every new ad set |
| Audience Expansion on by default | Unchecks after every format change or new ad set |
| Ad format change creates a new ad set | Renames, re-sets budget and placements on the new ad set |
| Calendar shows wrong month after typing date | Clicks nav arrow to correct month, then clicks the date |
| "Save objective" dialog on Next | Clicks Save to confirm |

## Targeting options

**Job Functions** (search by name in the targeting UI):
- Finance, Operations, Information Technology, Marketing, Business Development, etc.

**Company Industries** (search by name):
- Financial Services, Banking, Venture Capital, Software, etc.

**Member Interests** (search by name):
- Finance & Economy, Fintech, Cryptocurrency, etc.

## Dry run

Claude builds the ad set to **Draft** status and shows you a summary before activating. To request a dry run:

```
Build the LinkedIn ad set but leave it in Draft — don't activate.
```

## Validated results

- CTR: 3.9–4.6% (platform average: ~0.44%)
- CPE: $0.39–$0.79
- Platforms: LinkedIn Campaign Manager (React UI, as of July 2026)
