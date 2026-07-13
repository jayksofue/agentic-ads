---
name: agentic-ads-linkedin
description: Deploy LinkedIn ad campaigns via browser automation of Campaign Manager. Creates as Draft, activates only on explicit confirmation. LinkedIn's hierarchy is Campaign Group → Campaign → Ad (no "ad set" object). Use when working with LinkedIn ads specifically; loaded by the parent agentic-ads skill.
---

# LinkedIn Ads — Skill

Claude runs LinkedIn campaigns two ways — via browser automation of Campaign Manager (the default; live-launched at Eco) or via the LinkedIn Marketing API (documented but not yet used here). Both leave campaigns in **Draft** state until you explicitly say "launch."

> **QA status (2026-04, ongoing use): ✅ Method 1 (browser) production-ready.** Deployed at Eco: 3 campaigns across US/CA/AU, Singapore, and UK launched in a single session. Live results: CTR 3.9–4.6% (platform average ~0.44%), CPE $0.39–$0.79. LAN and Audience Expansion off enforcement, nativeSetter pattern for React inputs, and the "ad format change creates a new campaign" workaround are all covered by that live experience.
>
> **QA status Method 2 (LinkedIn Marketing API): 📝 Documented, not yet exercised.** LinkedIn's Marketing API requires developer-app approval (Marketing Developer Platform program) and OAuth2 with `r_ads`, `w_ads`, `r_ads_reporting` scopes. Field names below are from LinkedIn's public API reference; verify against a sandbox call before shipping automation.

## LinkedIn's object hierarchy (matters for terminology)

```
Ad Account
└── Campaign Group   ← equivalent to a Meta campaign (top-level container)
    └── Campaign     ← equivalent to a Meta ad set (targeting + budget + placements)
        └── Ad       ← the creative
```

Note: there is **no "ad set" object on LinkedIn** — the object that holds targeting and budget is called a **Campaign**, and it sits under a **Campaign Group**. If you're used to Meta's naming, mentally translate "ad set" → "campaign" and "campaign" → "campaign group" for the LinkedIn UI/API.

---

## Method 1: Browser automation (default; no API required)

Claude drives LinkedIn Campaign Manager directly in Chrome — same pattern as the other browser-primary platforms.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, then enable under Claude Code Settings → MCP → Browser. This is what lets Claude drive the Campaign Manager UI.
2. **Claude Code** — [claude.ai/code](https://claude.ai/code)
3. **Logged into LinkedIn Campaign Manager in Chrome** — navigate to [linkedin.com/campaignmanager](https://linkedin.com/campaignmanager) before starting
4. **Account ID** — find it in the Campaign Manager URL: `/accounts/{ID}/`
5. **Payment method on file** — LinkedIn requires a card added to the ad account before any campaign can leave Draft (the platform blocks activation otherwise; Draft creation itself is free)

### What Claude does

Claude navigates Campaign Manager, fills in every field for the new **Campaign** (LinkedIn's targeting+budget object), and attaches the creative. Before activating, it:
- Leaves the Campaign as **Draft** (never billable — LinkedIn's equivalent of Meta's PAUSED)
- Shows you a summary for review
- Activates only on your explicit "yes, launch" — never on its own initiative
- Can delete the Draft after QA if you ask

### Campaign setup — what Claude will ask you

When you start, Claude walks you through each decision. Here's what each option means:

---

### Ad format

| Format | What it is | Best for |
|---|---|---|
| **Document** | A multi-page PDF or carousel that users can scroll through in-feed | Lead gen, thought leadership, reports |
| **Single image** | One static image with headline + copy | Brand awareness, simple CTAs |
| **Video** | Autoplaying video in the feed | Storytelling, demos, product walkthroughs |

> **Recommendation:** Document ads consistently outperform single image for B2B engagement. If you have a report, deck, or guide — use Document.

---

### Audience targeting

LinkedIn lets you layer multiple targeting dimensions. Claude will ask what you know about your ideal customer:

**Job Function** — the department they work in
- Examples: Finance, Operations, Information Technology, Marketing, Business Development, Legal

**Job Title** — their specific role (more precise, smaller audience)
- Examples: CFO, Head of Treasury, VP Finance, Controller

**Seniority** — their level in the org
- Examples: Director, VP, C-Suite, Manager, Senior

**Company Industry** — the sector their company operates in
- Examples: Financial Services, Banking, Venture Capital, Software, Fintech

**Member Interests** — topics they engage with on LinkedIn
- Examples: Finance & Economy, Fintech, Cryptocurrency, Business Strategy

**Company Size** — number of employees
- Examples: 51–200, 201–500, 1,001–5,000, 10,001+

> **Tip:** Narrower targeting = higher CPE but better quality leads. Broader = cheaper but more noise. Start specific, then widen if volume is too low.

---

### LAN (LinkedIn Audience Network)

**What it is:** LinkedIn's extended ad network — your ad gets shown on third-party websites and apps outside of LinkedIn.com itself.

**Why we turn it off:** LAN placements are lower quality. Your ad appears on random third-party sites where engagement is shallow, inflating impression counts and wasting budget. Keeping it off means 100% of your spend goes to LinkedIn's actual feed.

**Default:** LinkedIn turns this ON automatically. Claude always turns it off unless you explicitly ask for it.

---

### Audience Expansion

**What it is:** LinkedIn's setting that automatically widens your audience beyond what you defined — showing your ad to people it thinks are "similar" to your targets.

**Why we turn it off:** You lose control over who sees the ad. LinkedIn's expansion logic often drifts into lower-quality segments. Your targeting decisions stop meaning anything.

**Default:** LinkedIn turns this ON automatically. Claude always turns it off unless you explicitly ask for it.

---

### Budget type

| Type | How it works | Best for |
|---|---|---|
| **Lifetime** | Fixed total spend over the campaign flight, LinkedIn paces it | Short campaigns, fixed budget, most control |
| **Daily** | Spend up to $X per day, runs indefinitely until you pause | Always-on campaigns |

> **Recommendation:** Use lifetime budget for campaigns with a defined end date. It gives LinkedIn flexibility to pace spend on your best days.

---

### Bidding

LinkedIn defaults to **Maximum Delivery** (automated bidding — LinkedIn spends your budget to get the most results at whatever the market rate is). This is fine for most campaigns.

If you want to cap what you pay per engagement, you can set a **Manual CPC/CPM bid** — but Maximum Delivery usually outperforms manual bidding unless you have strong historical data.

---

## Ad copy

Before attaching a creative, Claude checks or generates copy. See [copy/SKILL.md](../copy/SKILL.md) for the full process.

**If you have copy:** paste it in the prompt — Claude checks it for slop and character limits before proceeding.

**If you don't have copy:** Claude generates 2–3 variants using the StoryBrand framework (problem → solution → CTA), runs the anti-slop check, and presents the cleanest option for your approval before attaching.

### LinkedIn sponsored-content character limits

**Hard limits (LinkedIn will reject the ad past these):**
- **Intro text (`introText`):** 600 characters
- **Headline (`headline`):** 200 characters
- **Description (`description`, when shown):** varies by format

**Soft targets (avoids UI truncation in the feed):**
- **Intro text:** aim for 150 characters — after that LinkedIn collapses the remainder behind "…see more"
- **Headline:** aim for 70 characters — beyond that risks truncation on smaller screens

**CTA button:** LinkedIn CTAs are **preset dropdown options**, not free text. Choose from: `Apply`, `Download`, `Get quote`, `Learn more`, `Sign up`, `Subscribe`, `Register`, `Join`, `Attend`, `Request demo`, `View more` (exact list depends on ad format). Do not invent CTA text.

Other formats have their own limits — Conversation ads: 8000-char message body; Message ads: 60-char subject + 1500-char body + 25-char CTA text; Text ads: 25-char headline + 75-char description; Dynamic ads: 50-char headline + 70-char description + 18-char CTA. Ask Claude if you need format-specific limits.

---

## Campaign setup prompt

Once you know your answers, paste a prompt like:

```
Set up a LinkedIn campaign in Campaign Manager — leave it as Draft, don't launch.
- Account ID: [your account ID]
- Campaign group: [name or ID]
- Ad format: Document
- Audience: CFOs, VPs of Finance, and Treasurers in Financial Services and Banking. US, UK, Singapore.
- Seniority: Director, VP, C-Suite
- Budget: $500 lifetime, July 1–31
- Creative: [post URL or creative ID]
- CTA (preset): Learn more
- LAN: off
- Audience expansion: off
```

Claude builds it, hits Save as Draft, and reports back — nothing goes live without your explicit "yes, launch" as a follow-up.

---

## Key gotchas Claude handles automatically

| Issue | Fix |
|---|---|
| LinkedIn Audience Network on by default | Unchecks after every new campaign creation |
| Audience Expansion on by default | Unchecks after every format change or new campaign |
| Ad format change creates a new campaign object | Renames, re-sets budget, placements, AND re-adds all audience attributes on the new one |
| Audience attributes below the fold | Scroll down past the language setting on the details page before clicking Next — Job Function/Seniority/Interests are in the same section as locations |
| "C-Suite" not found | LinkedIn's seniority label is "CXO" — use that instead |
| Calendar shows wrong month after typing date | Clicks nav arrow to correct month, then clicks the date |
| "Save objective" dialog on Next | Clicks Save to confirm |
| Inner panel won't scroll with mouse | Use JS `document.querySelector('._container_x5gf48').scrollTop += 400` to scroll the content panel |
| **React inputs ignore `.value =` assignment** | Uses **nativeSetter** pattern: get the descriptor with `Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype, 'value')` and call `.set.call(inputEl, newValue)`, then dispatch an `input` event bubble. This is the same pattern used by Meta, X, and Google skills for controlled React inputs — see gotchas rows there. |
| Campaign auto-saves as Draft | Confirms Draft state in the URL/status pill before proceeding; only clicks Publish when you explicitly confirm |
| Preset CTA dropdown, not free text | Selects from LinkedIn's preset CTA list; does not attempt to type custom CTA text |

---

## Dry run

Claude builds the Campaign to **Draft** status and shows you a summary before activating. To request an explicit dry run:

```
Build the LinkedIn campaign but leave it in Draft — don't activate yet.
```

Draft = LinkedIn's equivalent of Meta's PAUSED. The Campaign exists, targeting is saved, budget is set, but LinkedIn will not bill or serve it. Review the Draft in Campaign Manager, then tell Claude to launch when ready.

---

## Delete after QA

To remove a Draft (or paused) campaign:

```
Delete the LinkedIn Draft campaign we just created — campaign name: [name]
```

Claude:
1. Navigates to the account's Campaigns view
2. Reads back the campaign name from the row to confirm it's the intended QA campaign (LinkedIn's row-checkbox flow doesn't disambiguate by name in the confirm dialog, so Claude verifies before clicking)
3. Selects the row's checkbox → **Actions → Delete** → confirm

Drafts and paused campaigns can be deleted freely. **Active** campaigns must be paused first (LinkedIn blocks direct delete on live ones).

---

## Method 2: LinkedIn Marketing API (documented, not yet exercised)

Programmatic path — useful for bulk setup or CI/CD. Requires developer-app approval through LinkedIn's Marketing Developer Platform.

- **Docs:** [learn.microsoft.com/linkedin/marketing/](https://learn.microsoft.com/linkedin/marketing/)
- **Base URL:** `https://api.linkedin.com/rest/`
- **Auth:** OAuth2 with the `r_ads`, `w_ads`, `r_ads_reporting` scopes (three-legged flow). Access tokens last ~60 days; refresh tokens ~365 days.
- **Header requirement:** `LinkedIn-Version: 202XXX` (monthly-versioned API — must be set explicitly on every request).

### Prerequisites

- LinkedIn developer app approved for the **Marketing Developer Platform** (approval can take weeks; there's no self-serve tier for the write scopes)
- OAuth2 credentials (client ID + secret) and a refresh token obtained via the three-legged flow
- Ad account URN (`urn:li:sponsoredAccount:{id}`)

Set credentials in your environment (read from a secret manager, not paste — history persists):

```
LINKEDIN_CLIENT_ID=...
LINKEDIN_CLIENT_SECRET=...
LINKEDIN_ACCESS_TOKEN=...
LINKEDIN_REFRESH_TOKEN=...
LINKEDIN_AD_ACCOUNT_ID=...
```

### Object structure via the API

- `adCampaignGroups` — top-level containers
- `adCampaigns` — the targeting+budget object (Meta's "ad set" equivalent)
- `adCreatives` — the actual ad content, referenced by campaigns

Create in that order. The paused-state field on both `adCampaigns` and `adCampaignGroups` is `status` with values `DRAFT`, `ACTIVE`, `PAUSED`, `ARCHIVED`, `COMPLETED`, `CANCELED`, `PENDING_DELETION`, `REMOVED`. **Create with `status: "DRAFT"` for QA** — the object exists but LinkedIn will not serve or bill.

### Dry run

LinkedIn's Marketing API has **no `validate_only` flag** (as of 2026). The QA path is create-with-`status: DRAFT`, verify by GET, then set `status: REMOVED` to clean up. Same shape as Reddit / TikTok's dry-run pattern.

### Delete after QA

```
PATCH https://api.linkedin.com/rest/adCampaigns/{id}
{ "patch": { "$set": { "status": "REMOVED" } } }
```

Or `DELETE https://api.linkedin.com/rest/adCampaigns/{id}` if the account permits direct deletes on drafts.

### ⚠️ Not yet live-verified

Field names, endpoint paths, and status enum values above come from LinkedIn's public docs; treat them as accurate but unverified from this environment. Confirm against a sandbox call on the developer test-tier before shipping automation.

---

## Validated live results (Method 1)

- Deployed at [Eco](https://eco.com)
- **CTR:** 3.9–4.6% (LinkedIn platform average: ~0.44% — roughly 10× baseline)
- **CPE:** $0.39–$0.79
- **Scale:** 3 campaigns across US/CA/AU, Singapore, UK — launched in one session
