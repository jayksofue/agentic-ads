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

### Objective-specific behavior

| Objective | Modal on create | Campaign-level toggle | Ad Set: Conversion location default | Ad Set: blocker |
|---|---|---|---|---|
| **Engagement** | "Recommended setup" Advantage+ modal → dismiss OK | None | N/A | None |
| **Traffic** | "Choose a campaign setup" → select Manual traffic campaign → Continue | None | Message destinations (switch to Website) | None |
| **Leads** | "You no longer need to choose a campaign setup" info modal → OK | Advantage+ leads campaign ON | Instant forms | Must accept Lead Ads Terms of Service on the Facebook Page |
| **Sales** | "Malfunctioning browser extension" may appear → dismiss OK | Advantage+ sales campaign ON | Website | Requires a Meta Pixel (Dataset) with conversion events set up |

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| Objective picker ignores coordinate clicks | Find `input[type="radio"]` inside `[role="row"]` and dispatch `mousedown + mouseup + click` events |
| "Recommended setup" Advantage+ modal on load (Engagement) | Dismisses by clicking OK |
| Traffic objective: "Message destinations" pre-selected | Switch to Website (or App/Calls) by dispatching mousedown+mouseup+click on the radio input |
| Traffic objective: "Choose a campaign setup" modal | Select "Manual traffic campaign" radio → click Continue |
| Audience Network on by default | **Engagement**: Placements → "Show more settings" → Placement controls → uncheck "Apps and sites". **Traffic/Sales (Advantage+ placements mode)**: no inline toggle — must switch to manual placements first via the ✎ icon |
| Advantage+ audience | "Switch to original audience options" link appears at bottom of Audience section — click it to use manual targeting |
| Dynamic creative deprecated | Meta removed the toggle — use up to 10 assets in a single image/video ad instead |
| Ad Identity: Threads + WhatsApp fields | Traffic and Sales objectives show Threads profile and WhatsApp phone number fields in the Ad level Identity section — not present in Engagement |
| WhatsApp Status placement included by default | Traffic/Website mode adds WhatsApp Status as a placement by default — cannot be removed inline in Advantage+ mode |
| Budget type resets on objective change | Re-enters budget after any objective switch |
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern: `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value').set` |
| "Publish" vs draft | Campaign auto-saves as draft — only click Publish when you confirm launch |
| Coordinate click scaling | Screenshot px ≈ CSS px × 0.61 (viewport is 2542 CSS px wide on a 1552px screenshot) |
| "Malfunctioning browser extension" warning | Meta detects the Claude for Chrome extension — dismiss with OK, then continue normally |
| Delete draft campaign | Navigate to campaign list → "Discard drafts" top-right (discards all unsaved changes in the account) |
| Row-level `...` dropdown | Hover campaign row → last button (aria-label contains zero-width space: "Open Dropdown​") → shows Analyze / View history / Rules — no Delete option here |
| Leads: Terms of Service blocker | Page owner must accept Facebook's Lead Generation Terms of Service before lead ads can run — Claude flags this and shows the "View terms" button |
| Sales: Pixel required | Sales campaigns require a Dataset (Meta Pixel) with at least one conversion event configured — Claude shows "Set up conversion event" if none exist |

### Dry run prompt

```
Set up a Meta campaign in Ads Manager — leave it in draft, don't publish.
Objective: Engagement / Traffic / Leads / Sales
Audience: [description — age, location, interests]
Budget: $[amount] daily or lifetime, [dates]
Creative: [image URL or post URL]
```

For **Sales** campaigns, also provide:
- Pixel name (Meta will auto-select if only one exists)
- Conversion event (Purchase, AddToCart, Lead, etc.)

For **Leads** campaigns, confirm the Facebook Page has accepted Lead Generation Terms of Service first.

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
