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

## Method 2: Marketing API via the `meta-ads` CLI

Claude calls the Marketing API through the `meta-ads` command-line tool. Faster for bulk setup, no UI interaction.

> **QA status (2026-07-02): NOT confirmed from claude.ai.**
> The live dry-run → create-paused → delete cycle could not be run in a non-interactive claude.ai session, because no `META_ACCESS_TOKEN` is available there to authenticate against Meta. The commands below are verified against the real `meta-ads` CLI v0.18.1 `--help` output, but the end-to-end API round-trip has only been exercised from a terminal with a configured token, not from claude.ai. A non-interactive session cannot complete the OAuth/token step on its own.
>
> Earlier drafts of this section referenced `@anthropic-ai/mcp-server-meta` and `cli-meta-ads`. Neither exists on npm anymore (`@anthropic-ai/mcp-server-meta` returns 404; `cli-meta-ads` was unpublished 2026-05-28). Use the `meta-ads` package instead.

### Prerequisites

1. **Install the CLI** (npm package `meta-ads`, actively maintained, v0.18.1+):

   ```bash
   npm install -g meta-ads
   # or run ad-hoc without installing:
   npx -y meta-ads@latest --help
   ```

   Verify with `meta-ads --version`.

2. **Authenticate.** Interactive OAuth or a pasted token:

   ```bash
   meta-ads auth login      # OAuth2, or paste an access token
   meta-ads auth status     # confirm you're authenticated
   ```

   Non-interactive (a terminal or CI where a token is already exchanged):

   ```bash
   meta-ads setup --non-interactive --token "$META_ACCESS_TOKEN" --account-id act_XXXXXXXXXX
   ```

   The token needs the `ads_management`, `ads_read`, and `business_management` scopes and expires roughly every 60 days.

3. **Ad Account ID** — format: `act_XXXXXXXXXX`

### What Claude does via API

Claude creates the campaign and ad set, both PAUSED. The structure matches Ads Manager:
- Campaign: objective, name, status = PAUSED, budget
- Ad Set: targeting, optimization goal, billing event

```bash
meta-ads campaigns create \
  --account-id act_XXXXXXXXXX \
  --name "QA_Engagement_Jul2026" \
  --objective OUTCOME_ENGAGEMENT \
  --status PAUSED \
  --daily-budget 1000
```

**Key CLI behaviors (verified against meta-ads v0.18.1):**
- Budget is passed in **cents**: `$10 daily = --daily-budget 1000`
- Objective uses the `OUTCOME_*` enum: `OUTCOME_AWARENESS`, `OUTCOME_TRAFFIC`, `OUTCOME_ENGAGEMENT`, `OUTCOME_LEADS`, `OUTCOME_APP_PROMOTION`, `OUTCOME_SALES`. Engagement = `OUTCOME_ENGAGEMENT`.
- `--status` defaults to `PAUSED` on create, so nothing serves until you flip it to `ACTIVE`.
- `--dry-run` prints the exact request without executing it, so nothing is created (see below).
- Advantage+ Audience and Dynamic Creative are controlled through the ad set targeting/creative spec, not campaign flags. Do not assume a default; set them explicitly and re-read with `meta-ads adsets get` to confirm.

### Campaign setup prompt

```
Create a Meta ads campaign via the Marketing API:
- Ad account: act_[your account ID]
- Objective: OUTCOME_AWARENESS / OUTCOME_ENGAGEMENT / OUTCOME_TRAFFIC / OUTCOME_SALES
- Audience: [age range, locations, interests, custom audiences]
- Budget: $[amount] daily or lifetime
- Start/end: [dates]
- Creative: [image URL or video URL + copy]
- Placement: Feed only (no Audience Network)
- Dry run: yes/no
```

### Dry run mode

The CLI supports `--dry-run` on `create` and `update`. It prints the exact API request that would be sent and does **not** execute it, so nothing is created. This is a true no-op.

```bash
meta-ads campaigns create \
  --account-id act_XXXXXXXXXX \
  --name "QA_Engagement_Jul2026" \
  --objective OUTCOME_ENGAGEMENT \
  --status PAUSED \
  --daily-budget 1000 \
  --dry-run
```

Claude returns the request payload it would send. Drop `--dry-run` to actually create the (still PAUSED) objects.

### Delete after QA

The CLI has no `delete` subcommand. Set status to `DELETED` via `update` (`--force` skips the destructive-change confirmation). Delete the ad set first, then the campaign:

```bash
meta-ads adsets update    --adset-id <ADSET_ID>       --status DELETED --force
meta-ads campaigns update --campaign-id <CAMPAIGN_ID> --status DELETED --force
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
