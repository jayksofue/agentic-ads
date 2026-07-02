# Meta Ads — Skill

Claude can run Meta campaigns two ways — via the Marketing API (no UI needed) or via browser automation (no API setup needed). Both support draft/paused mode so you can review before spending.

---

## Method 1: Browser automation (no API required)

Same pattern as the LinkedIn skill — Claude drives Meta Ads Manager directly in Chrome.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [Meta Ads Manager](https://adsmanager.facebook.com) in Chrome before starting
3. Your Ad Account ID — found in the URL: `act_XXXXXXXXXX`

### What Claude does

Claude navigates Ads Manager, fills in campaign objective, audience, placements, budget, and creative. Before activating, it:
- Sets campaign status to **Paused** (draft equivalent — won't serve)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the draft campaign after QA if you ask

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| Audience Network on by default | Unchecks under Placements → Manual placements |
| Detailed Targeting Expansion on by default | Unchecks before saving ad set |
| Budget type resets on objective change | Re-enters budget after any objective switch |
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern same as LinkedIn |
| "Publish" vs "Save draft" | Always clicks Save draft until you confirm launch |

### Dry run prompt

```
Set up a Meta campaign in Ads Manager — leave it Paused, don't publish.
Objective: Engagement
Audience: Finance decision-makers in the US, 30–55
Budget: $500 lifetime, July 1–31
Creative: [image URL or post URL]
```

### Delete after QA

```
Delete the draft Meta campaign we just created — campaign name: [name]
```

Claude navigates to the campaign, selects it, and deletes it from the campaign list.

---

## Method 2: Marketing API via Meta MCP

Claude calls the API directly — faster for bulk setup, no UI interaction.

### Prerequisites

1. **Install the Meta MCP**

   ```bash
   npx @anthropic-ai/mcp-server-meta
   ```

   Or add to your Claude Code MCP config:

   ```json
   {
     "mcpServers": {
       "meta": {
         "command": "npx",
         "args": ["-y", "@anthropic-ai/mcp-server-meta"]
       }
     }
   }
   ```

2. **Authenticate** — on first run, Claude prompts you to connect your Meta Business account via OAuth. Approve in the Meta dialog — Claude never sees your password.

3. **Ad Account ID** — format: `act_XXXXXXXXXX`

### Campaign setup prompt

```
Create a Meta ads campaign:
- Ad account: act_[your account ID]
- Objective: AWARENESS / ENGAGEMENT / TRAFFIC / CONVERSIONS
- Audience: [age range, locations, interests, custom audiences]
- Budget: $[amount] daily or lifetime
- Start/end: [dates]
- Creative: [image URL or video URL + copy]
- Placement: Feed only (no Audience Network)
- Dry run: yes/no
```

### Dry run mode

Meta's API supports `execution_options: { validate_only: true }` — Claude submits the full campaign payload for validation without creating anything.

```
Create a Meta campaign for [audience] — validate only, don't launch yet.
```

Claude returns: full campaign JSON, API validation result, targeting reach estimate.

### Delete after QA

```python
# Claude calls the API directly:
DELETE /act_{ad_account_id}/campaigns?campaign_id={id}
```

### Key settings Claude enforces

- `special_ad_categories: []` — required if your ad is not in housing/credit/employment
- Audience Network placement: excluded by default (same as LAN off on LinkedIn)
- `bid_strategy: LOWEST_COST_WITHOUT_CAP` unless you specify a target CPA
- Campaign status: `PAUSED` until you explicitly confirm launch

### Scaling: multiple ad sets

```
Create 3 Meta ad sets under campaign [ID]:
- Ad set 1: US, 25–45, interest: fintech. $500 lifetime.
- Ad set 2: UK, 30–55, interest: banking. $300 lifetime.
- Ad set 3: Singapore, 25–50, lookalike of [customer list]. $400 lifetime.
```

Claude creates all three in sequence, confirms each before creating the next.
