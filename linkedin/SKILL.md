# LinkedIn Ads — Skill

Claude drives LinkedIn Campaign Manager directly via browser automation. No API key required.

## Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, then enable under Claude Code Settings → MCP → Browser. This is what lets Claude drive the Campaign Manager UI.
2. **Claude Code** — claude.ai/code
3. **Logged into LinkedIn Campaign Manager in Chrome** — navigate to linkedin.com/campaignmanager before starting
4. **Account ID** — find it in the Campaign Manager URL: `/accounts/{ID}/`

## How it works

Claude navigates Campaign Manager, fills in all fields, and launches the ad set. You confirm before anything goes live.

## Campaign setup — what Claude will ask you

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

## Campaign setup prompt

Once you know your answers, paste a prompt like:

```
Launch a LinkedIn sponsored content campaign:
- Account ID: [your account ID]
- Campaign group: [name or ID]
- Ad format: Document
- Audience: CFOs, VPs of Finance, and Treasurers in Financial Services and Banking. US, UK, Singapore.
- Seniority: Director, VP, C-Suite
- Budget: $500 lifetime, July 1–31
- Creative: [post URL or creative ID]
- LAN: off
- Audience expansion: off
```

---

## Key gotchas Claude handles automatically

| Issue | Fix |
|---|---|
| LinkedIn Audience Network on by default | Unchecks after every new ad set creation |
| Audience Expansion on by default | Unchecks after every format change or new ad set |
| Ad format change creates a new ad set | Renames, re-sets budget, placements, AND re-adds all audience attributes on the new one |
| Audience attributes below the fold | Scroll down past the language setting on the details page before clicking Next — Job Function/Seniority/Interests are in the same section as locations |
| "C-Suite" not found | LinkedIn's seniority label is "CXO" — use that instead |
| Calendar shows wrong month after typing date | Clicks nav arrow to correct month, then clicks the date |
| "Save objective" dialog on Next | Clicks Save to confirm |
| Inner panel won't scroll with mouse | Use JS `document.querySelector('._container_x5gf48').scrollTop += 400` to scroll the content panel |

---

## Dry run

Claude builds the ad set to **Draft** status and shows you a summary before activating. To request a dry run:

```
Build the LinkedIn ad set but leave it in Draft — don't activate yet.
```

---

## Validated results

- CTR: 3.9–4.6% (platform average: ~0.44%)
- CPE: $0.39–$0.79
- 3 ad sets across 3 regions launched in one session
