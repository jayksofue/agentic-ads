---
name: agentic-ads-meta
description: Launch Meta (Facebook / Instagram) ad campaigns in Ads Manager, either through the browser or through Meta's Marketing API. Every campaign is created as Paused — nothing goes live without an explicit "launch" from the user. Handles Meta's Campaign → Ad Set → Ad hierarchy, budget-optimization modes (CBO / ABO), Advantage+ audience toggles, and regulated-category declarations for financial services / crypto. Use this when launching Meta ads specifically; loaded automatically by the parent agentic-ads skill.
---

# Meta Ads — Skill

Two ways to launch Meta campaigns:

- **Through the browser** — Claude drives Meta Ads Manager directly in Chrome. Easiest to set up (no developer account needed), and works for everyone with a Meta ad account.
- **Through Meta's Marketing API command-line tool** — faster for bulk work and repeatable campaigns. Requires a one-time setup of a Meta developer app + an access token.

Both paths leave the campaign Paused for review; neither runs anything live until you say so.

---

## Method 1: Through the browser (no API setup needed)

Claude drives Meta Ads Manager directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [Meta Ads Manager](https://adsmanager.facebook.com) in Chrome before starting
3. Your Ad Account ID — found in the URL: `act_XXXXXXXXXX`
4. **Payment method on file** in the Meta ad account (Business Settings → Payments). Meta will block activation without one, though PAUSED campaign creation itself works.

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
| Delete draft campaign | ⚠️ **Destructive/account-wide**: the top-right "Discard drafts" button discards **every unsaved draft in the account**, not just the QA campaign. Do NOT use for cleanup unless the account has no other in-progress drafts. Safe path is Method 2's `meta ads campaign delete <ID> --force` (deletes only the named campaign and its children). Requires an explicit human "yes, discard all drafts" if this button is ever the only route. |
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

Method 1 (browser) has **no safe per-campaign delete for drafts** — the row `…` dropdown has no Delete option (see gotchas table), and the top-right "Discard drafts" is account-wide destructive. **Route Method 1 cleanup through Method 2's CLI delete** instead:

```
meta ads campaign delete <CAMPAIGN_ID> --force
```

If Method 2 isn't set up on the account, either wait for the draft to age off or manually publish it PAUSED first (which then exposes a normal Remove control) — never rely on "Discard drafts" for QA cleanup because it wipes every other unsaved draft in the account too.

---

## Method 2: Meta's Marketing API command-line tool

For faster, repeatable setups (or bulk work across many campaigns), Claude uses Meta's Marketing API through a command-line tool called [`meta-ads`](https://pypi.org/project/meta-ads/). This is the tool referenced in Meta's April 2026 developer blog post [Introducing Ads CLI](https://developers.facebook.com/blog/post/2026/04/29/introducing-ads-cli/). The commands match Meta's blog examples exactly, though the package listing on PyPI doesn't publicly attribute it to Meta (blank author field, proprietary license) — so this doc treats it as "referenced by Meta's official blog" rather than formally Meta-attributed. If you need signed provenance, there's also a third-party alternative documented at the bottom of this file with a public GitHub repo.

> **Status: tool verified, one live end-to-end run pending.** The command-line tool was installed and exercised directly (every command, flag, and campaign-setting name was pulled from the tool itself, not from docs). The one thing not re-run: the full paused-create → delete cycle with a fresh access token. The access token from the earlier test session had expired by the time the re-run was attempted. Once a fresh token is provided, that final check runs and the status flips to ✅.
>
> An earlier full end-to-end run **did** pass through a third-party alternative tool (documented at the bottom of this file): a campaign was created Paused, then deleted, on a real Meta ad account. This doc recommends Meta's official-blog-referenced tool as the primary path because third-party npm packages can vanish without notice (that's exactly what happened to two earlier alternatives — `cli-meta-ads` was unpublished from npm, and `@anthropic-ai/mcp-server-meta` was never actually published at all).

### Prerequisites

1. **Install the CLI** (requires **Python 3.12+**):

   ```bash
   pip install meta-ads        # or:  uv tool install meta-ads
   meta --version              # -> meta, version 1.1.0
   ```

2. **Authenticate** via the `ACCESS_TOKEN` environment variable (token needs `ads_management`, `ads_read`, `business_management` scopes; short-lived tokens expire in ~1–24h, long-lived ~60 days; System User tokens don't expire — see the token-refresh subsection below).

   Read the token from a file or password manager, not by pasting into your shell (shell history persists secrets):

   ```bash
   # Read from ~/.config/meta-ads-cli/config.json (populated by the npm fallback CLI's `meta-ads auth login`)
   export ACCESS_TOKEN="$(python3 -c "import json;print(json.load(open('$HOME/.config/meta-ads-cli/config.json'))['auth']['access_token'])")"
   # or from macOS Keychain:
   # export ACCESS_TOKEN="$(security find-generic-password -s meta-ads-token -w)"
   meta auth status            # -> Authenticated (token: EAA...) — reports presence, NOT validity
   ```

   > `meta auth status` only checks that a token is *present*, not that it is still valid. An expired token still reports "Authenticated" and fails at the first API call with `API error (190): Session has expired`. Treat a real command as the true auth check.

3. **Ad account:** pass `--ad-account-id act_XXXX` on the `ads` group, or set `AD_ACCOUNT_ID=act_XXXX`.

### What Claude does via API

Claude creates the campaign and ad set, both PAUSED. Note the **flag placement** (they are grouped, not per-subcommand):
- `-o/--output {table,json,plain}` goes on the top-level `meta` command, **before** `ads`.
- `--ad-account-id` goes on the `ads` group (`meta ads --ad-account-id … campaign create`), or use the `AD_ACCOUNT_ID` env var.

```bash
export AD_ACCOUNT_ID=act_XXXXXXXXXX
meta -o json ads campaign create \
  --name "QA_Engagement_Jul2026" \
  --objective OUTCOME_ENGAGEMENT \
  --daily-budget 1000 \
  --status PAUSED
# then, with the returned CAMPAIGN_ID:
meta ads adset create <CAMPAIGN_ID> \
  --name "US_Fintech_25-45" \
  --optimization-goal REACH --billing-event IMPRESSIONS \
  --targeting @targeting.json
```

**Key CLI behaviors (verified against meta-ads 1.1.0):**
- Budget is passed in **cents**: `$10 daily = --daily-budget 1000`.
- Objective enum: `OUTCOME_APP_PROMOTION`, `OUTCOME_AWARENESS`, `OUTCOME_ENGAGEMENT`, `OUTCOME_LEADS`, `OUTCOME_SALES`, `OUTCOME_TRAFFIC`. Engagement = `OUTCOME_ENGAGEMENT`.
- `--status` defaults to `PAUSED` on create — nothing serves until you set `ACTIVE`.
- **No `--dry-run` flag.** Unlike the third-party npm tool, the PyPI CLI has no built-in validate/no-op mode. The QA path is: create PAUSED (a real object that can't serve), verify, then delete.
- **Where the budget lives (CBO vs. ABO).** Meta lets you set the budget in two places:
  - **Campaign Budget Optimization (CBO)** — put the budget on the *campaign*, and Meta distributes it across the ad sets automatically. Set `--daily-budget` or `--lifetime-budget` on the campaign; omit budget on the ad sets.
  - **Ad-set Budget Optimization (ABO)** — leave the campaign budget off, and set a budget per ad set instead. Better when you want strict control per audience.
  - Don't set both — Meta rejects it.
- Ad-set targeting: `--targeting-countries US` is a shortcut for geo only; for age + interests + placement + Advantage+-off, pass the full spec via `--targeting @file.json` (see JSON example below). The shortcut path does NOT respect the `advantage_audience: 0` or placement settings — those only apply in the JSON path (or you can pass `--no-advantage-audience` on the ad-set create for that one setting).
- **Placement restrictions must be inside the targeting JSON**: the PyPI CLI has no placement flag. To restrict to Facebook Feed only (no Audience Network / Reels / Stories / Instagram): add `publisher_platforms: ["facebook"]` and `facebook_positions: ["feed"]` to `--targeting @file.json`. Without these, ad sets default to Advantage+ placements — including Audience Network.

**Example `targeting.json` with Advantage+ off and placement restricted:**
```json
{
  "age_min": 25, "age_max": 55,
  "geo_locations": {"countries": ["US"]},
  "flexible_spec": [{"interests": [{"id": "6003107902433", "name": "Financial technology (fintech)"}]}],
  "publisher_platforms": ["facebook"],
  "facebook_positions": ["feed"],
  "device_platforms": ["mobile", "desktop"],
  "targeting_automation": {"advantage_audience": 0}
}
```

### Campaign setup prompt

```
Create a Meta ads campaign via the meta ads CLI:
- Ad account: act_[your account ID]
- Objective: OUTCOME_AWARENESS / OUTCOME_ENGAGEMENT / OUTCOME_TRAFFIC / OUTCOME_SALES
- Audience: [age range, locations, interests, custom audiences]
- Budget: $[amount] daily or lifetime
- Placement: pass as targeting JSON (publisher_platforms + facebook_positions) — the CLI has no placement flag
- Special-ad-category: [FINANCIAL_PRODUCTS_SERVICES for fintech/finance ads; HOUSING/EMPLOYMENT/CREDIT/ISSUES_ELECTIONS_POLITICS/ONLINE_GAMBLING_AND_GAMING if applicable; otherwise omit]
```

### Delete after QA

The CLI has a real `delete` that **cascades** — removing the campaign also removes its child ad sets and ads:

```bash
# 1. Confirm you have the right ID + it's PAUSED
meta ads campaign get <CAMPAIGN_ID>          # -> verify name and status: PAUSED

# 2. Delete (Claude echoes the name back to the operator before running --force)
meta ads campaign delete <CAMPAIGN_ID>       # interactive: prompts to confirm
# or, only after echoing the campaign name and confirming it's the QA one:
meta ads campaign delete <CAMPAIGN_ID> --force
```

⚠️ **`--force` is irreversible and cascades.** Claude requires an ID-match guard: read back the campaign name + status, echo them to the operator, and only run `--force` after the operator confirms it's the intended QA campaign (not a real one with a similar ID).

### Key settings Claude enforces

- **Special-ad-categories** (`--special-ad-categories`): set for any regulated category. Real enum from the CLI: `HOUSING`, `EMPLOYMENT`, `CREDIT`, **`FINANCIAL_PRODUCTS_SERVICES`**, `ISSUES_ELECTIONS_POLITICS`, `ONLINE_GAMBLING_AND_GAMING`. **Financial-services and crypto/fintech ads must declare `FINANCIAL_PRODUCTS_SERVICES`** or Meta will disapprove them — this applies to almost every Eco-style campaign. Omit only when the ad is genuinely outside all regulated categories.
- Campaign status: `PAUSED` until you explicitly confirm launch.
- Advantage+ audience expansion off unless asked (`targeting_automation.advantage_audience: 0` in the targeting JSON, or `--no-advantage-audience` on the ad-set create).
- Placement restricted to Facebook Feed (or the placements you specify) via the targeting JSON; do not rely on Advantage+ defaults if you want to exclude Audience Network.

### Token refresh (long-lived tokens)

Access tokens generated from Graph API Explorer are short-lived (1–24h). For anything longer than a single session, exchange for a **long-lived (60-day) user token** or use a **System User token** (permanent for a Business Manager):

```bash
# Long-lived user token: exchange short-lived for 60-day
curl -sG "https://graph.facebook.com/v21.0/oauth/access_token" \
  --data-urlencode "grant_type=fb_exchange_token" \
  --data-urlencode "client_id=$FB_APP_ID" \
  --data-urlencode "client_secret=$FB_APP_SECRET" \
  --data-urlencode "fb_exchange_token=$SHORT_LIVED_TOKEN"
```

For automation, prefer a Business Manager **System User** token (never expires) — create under Business Settings → Users → System Users → Generate New Token, and grant it Ads Management + Read access on the ad account.

`meta auth status` reports token *presence*, not validity — always treat a real API call as the true auth check.

### Fallback: third-party npm `meta-ads` CLI

If Python 3.12+ isn't available, the third-party npm package `meta-ads` (by `gupsammy`, Node) is a documented alternative that was **live-verified end-to-end on 2026-07-04**. It has a `--dry-run` no-op the official CLI lacks. Command shape differs: `meta-ads campaigns create --account-id … --daily-budget 1000` (plural `campaigns`, hyphenated binary), auth via `meta-ads auth login --token …` (persists to `~/.config/meta-ads-cli/config.json`), delete via `meta-ads campaigns update --campaign-id … --status DELETED --force`. Ad-set gotcha there: a CBO campaign with no explicit bid strategy may reject the ad set with `Invalid parameter` (subcode `1815857`) until you add `--bid-amount <cents>`. Prefer the official CLI; treat this as the fallback.

### Scaling: multiple ad sets

```
Create 3 Meta ad sets under campaign [ID]:
- Ad set 1: US, 25–45, interest: fintech. $500 lifetime.
- Ad set 2: UK, 30–55, interest: banking. $300 lifetime.
- Ad set 3: Singapore, 25–50, lookalike of [customer list]. $400 lifetime.
```

Claude creates all three in sequence, confirms each before creating the next.

### Handling partial failures (orphan cleanup)

Multi-step create sequences (`campaign` → `adset` → `ad`) can fail partway — e.g. the ad set is rejected but the campaign was already created. Rules Claude follows:

1. Capture the parent object's ID immediately after each successful create (`meta ads campaign create -o json`).
2. If any child create fails, the campaign delete is cascade-safe: `meta ads campaign delete <CAMPAIGN_ID> --force` removes it and any child objects that did get created.
3. Never leave an activated campaign behind after a partial failure — even a PAUSED orphan can be turned live by mistake later.
4. If the CLI itself dies before the campaign ID is captured, `meta ads campaign list -o json | grep <name>` finds it by name; the QA name convention (`QA_TEST_…`) makes this reliable.

### Ad review SLA

Once activated, Meta ads enter review — typically 15 min to a few hours, but **crypto/fintech and other regulated categories can take 24h+** and may be rejected on discretionary grounds. Do not schedule launch windows that assume instant serving. Check status via `meta ads ad get <AD_ID>` — `IN_PROCESS` = under review, `ACTIVE` = serving, `DISAPPROVED` = rejected (readable reason in `issues_info`).
