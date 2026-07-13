---
name: agentic-ads-meta
description: Deploy Meta (Facebook/Instagram) ad campaigns via browser automation (ads.facebook.com) or the Marketing API. Two CLI paths: the anonymous PyPI `meta-ads` (binary `meta`, referenced in Meta's April 2026 Ads CLI blog post) and the third-party npm `meta-ads` fallback. Every create defaults to PAUSED. Use when working with Meta ads specifically; loaded by the parent agentic-ads skill.
---

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

## Method 2: Marketing API via the `meta-ads` PyPI CLI (`meta ads`)

Claude calls the Marketing API through the command-line tool published as [`meta-ads` on PyPI](https://pypi.org/project/meta-ads/) (binary `meta`, invoked as `meta ads …`). This is the CLI referenced in Meta's April 2026 developer-blog post [Introducing Ads CLI](https://developers.facebook.com/blog/post/2026/04/29/introducing-ads-cli/) — the blog is a verified public Meta post and the CLI's `meta ads campaign create --name … --objective OUTCOME_SALES --daily-budget 5000` matches the blog's examples verbatim, but the PyPI package's author/repo fields are blank (proprietary license, Cython-compiled `.so` modules). Treat it as **blog-referenced**, not formally attributed to Meta — it works, but if you need signed provenance, use the third-party npm `meta-ads` fallback below (also unofficial but with a public repo).

> **QA status (2026-07-10): tool verified; live create→delete re-run pending a fresh token.**
> The official CLI was installed and exercised directly: `meta --version` (1.1.0), `meta auth status`, and the full `campaign`/`adset`/`delete` command surface + flags + enums were confirmed from the binary (not from docs). Auth wiring works — the token was accepted; the only failure was the reused access token having **expired** (it was a short-lived Jul-4 token). The full paused-create → delete cycle will be re-confirmed once a fresh token is provided, then this banner flips to ✅.
>
> A prior end-to-end run **did** pass on 2026-07-04 using the *third-party* npm `meta-ads` CLI (see fallback below): campaign `52513568511696` + ad set `52513568917696` created PAUSED on `act_15495450`, then deleted. We are standardizing on the official Python CLI because the third-party npm tool can be unpublished without notice — exactly what happened to the earlier `cli-meta-ads` (unpublished 2026-05-28) and the never-real `@anthropic-ai/mcp-server-meta` (404).

### Prerequisites

1. **Install the CLI** (requires **Python 3.12+**):

   ```bash
   pip install meta-ads        # or:  uv tool install meta-ads
   meta --version              # -> meta, version 1.1.0
   ```

2. **Authenticate** via the `ACCESS_TOKEN` environment variable (token needs `ads_management`, `ads_read`, `business_management` scopes; short-lived tokens expire in ~1–24h, long-lived ~60 days):

   ```bash
   export ACCESS_TOKEN="EAA...your_token..."
   meta auth status            # -> Authenticated (token: EAA...) — presence only, not validity
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
- **No `--dry-run` flag.** Unlike the third-party npm tool, the official CLI has no built-in validate/no-op mode. The QA path is: create PAUSED (a real object that can't serve), verify, then delete.
- CBO vs ABO: put `--daily-budget`/`--lifetime-budget` on the **campaign** (CBO) and omit it on the ad set; or omit it on the campaign and set it per ad set (ABO). Don't set both.
- Ad-set targeting: `--targeting-countries US` is a shortcut for geo only; for age + interests pass the full spec via `--targeting @file.json` (e.g. `age_min`/`age_max`, `geo_locations`, `flexible_spec` interests, `targeting_automation.advantage_audience: 0`).

### Campaign setup prompt

```
Create a Meta ads campaign via the official meta ads CLI:
- Ad account: act_[your account ID]
- Objective: OUTCOME_AWARENESS / OUTCOME_ENGAGEMENT / OUTCOME_TRAFFIC / OUTCOME_SALES
- Audience: [age range, locations, interests, custom audiences]
- Budget: $[amount] daily or lifetime
- Placement: Feed only (no Audience Network)
```

### Delete after QA

The official CLI has a real `delete` that **cascades** (removes the campaign and all its child ad sets and ads) — no need to delete children first:

```bash
meta ads campaign delete <CAMPAIGN_ID> --force   # --force skips the confirm prompt
```

### Key settings Claude enforces

- `--special-ad-categories` set only for regulated ads (housing/credit/employment/politics); omit otherwise.
- Campaign status: `PAUSED` until you explicitly confirm launch.
- Advantage+ audience expansion off unless asked (`targeting_automation.advantage_audience: 0` in the targeting spec).

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
