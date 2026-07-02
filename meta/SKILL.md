# Meta Ads — Skill

Claude connects to Meta's Marketing API via the Meta MCP. No browser automation needed — Claude calls the API directly.

## Prerequisites

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

2. **Authenticate**

   On first run, Claude will prompt you to connect your Meta Business account via OAuth. Approve in the Meta dialog — Claude never sees your password.

3. **Get your Ad Account ID**

   Found in Meta Business Manager → Business Settings → Ad Accounts. Format: `act_XXXXXXXXXX`

## Campaign setup prompt

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

## Dry run mode

Meta's API supports `execution_options: { validate_only: true }` — Claude submits the full campaign payload for validation without creating anything or spending budget.

```
Create a Meta campaign for [audience] — validate only, don't launch yet.
```

Claude returns:
- Full campaign JSON that would be submitted
- API validation result (errors caught before launch)
- Summary of targeting reach estimate

## Key settings Claude enforces

- `special_ad_categories: []` — required if your ad is NOT in housing/credit/employment
- Audience Network placement: excluded by default (same as LAN off on LinkedIn)
- `bid_strategy: LOWEST_COST_WITHOUT_CAP` unless you specify a target CPA

## Scaling: multiple ad sets

```
Create 3 Meta ad sets under campaign [ID]:
- Ad set 1: US, 25–45, interest: fintech. $500 lifetime.
- Ad set 2: UK, 30–55, interest: banking. $300 lifetime.
- Ad set 3: Singapore, 25–50, lookalike of [customer list]. $400 lifetime.
```

Claude creates all three in sequence, confirms each before creating the next.
