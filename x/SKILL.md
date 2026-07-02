# X Ads — Skill

Claude can run X campaigns two ways — via browser automation (works today, no API approval needed) or via the X Ads API (requires separate approval). Both support paused/draft mode.

---

## Method 1: Browser automation (no API required)

Claude drives [ads.x.com](https://ads.x.com) directly in Chrome — same pattern as the LinkedIn skill.

### Prerequisites

1. **Claude for Chrome extension** — install from the Chrome Web Store, enable under Claude Code Settings → MCP → Browser
2. Log into [X Ads Manager](https://ads.x.com) in Chrome before starting
3. Your X Ads account ID — found in the URL after logging in

### What Claude does

Claude navigates X Ads Manager, creates a campaign, sets objective, audience, budget, and attaches a tweet or creates a new promoted-only post. Before activating, it:
- Sets campaign status to **Paused** (won't serve, won't spend)
- Shows you a summary for review
- Activates only on your explicit confirmation
- Can delete the paused campaign after QA if you ask

### Key gotchas Claude handles

| Issue | Fix |
|---|---|
| "Tailored audiences" opt-in prompt on first visit | Dismisses — uses interest/keyword targeting instead unless you have a list |
| Budget type defaults to daily | Switches to total budget if you specify a fixed amount |
| Tweet selection requires exact URL | Uses the post URL from your X profile or a promoted-only tweet |
| Promoted-only tweet not visible on timeline | Expected — it's ad-only by design |
| React inputs ignore `.value =` assignment | Uses nativeSetter pattern same as LinkedIn |

### Campaign setup prompt

```
Set up an X campaign in Ads Manager — leave it Paused, don't launch.
Objective: Engagement / Awareness / Website clicks / Followers
Tweet to promote: [tweet URL or "create a new promoted-only post"]
Audience: [keywords, interests, follower lookalikes of @handle, demographics]
Locations: [countries]
Budget: $[amount] total or daily
Start/end: [dates]
```

### Delete after QA

```
Delete the paused X campaign we just created — campaign name: [name]
```

Claude navigates to the campaign list, selects it, and deletes it.

---

## Method 2: X Ads API

The X Ads API is free once approved but requires a separate application from general X API access. Apply at [ads.x.com/help](https://ads.x.com/help). Approval typically takes a few weeks.

### Prerequisites

Once approved:
- A developer app with Ads API (Standard Access) enabled
- `TWITTER_ADS_ACCOUNT_ID` (in the URL at ads.x.com)
- Consumer key/secret + access token/secret

Set credentials in your environment:

```
TWITTER_CONSUMER_KEY=...
TWITTER_CONSUMER_SECRET=...
TWITTER_ACCESS_TOKEN=...
TWITTER_ACCESS_TOKEN_SECRET=...
TWITTER_ADS_ACCOUNT_ID=...
```

Optional: install the SDK:

```bash
pip install twitter-ads
```

### Campaign setup prompt

```
Create an X promoted post campaign:
- Account ID: [your account ID]
- Objective: ENGAGEMENT / AWARENESS / WEBSITE_CLICKS / FOLLOWERS
- Tweet to promote: [tweet URL or ID]
- Audience: [keywords, interests, follower lookalikes, demographics]
- Locations: [countries]
- Budget: $[amount] total or daily
- Start/end: [dates]
- Dry run: yes/no
```

### Dry run mode

X Ads API has no native `validate_only` flag. Claude generates a full structured preview instead:

```
Show me the X campaign payload — don't submit yet.
```

Claude returns: full API request body (campaign + line item + targeting + creative), estimated audience size, and explicit confirmation prompt before any call is made.

### Delete after QA

```
Delete the X campaign with ID [campaign_id] — API call only, don't confirm.
```

Claude calls `DELETE /1/accounts/{account_id}/campaigns/{campaign_id}`.

### Key settings Claude enforces

- `charge_by: ENGAGEMENT` — pay per engagement, not impression
- Campaign status: `PAUSED` until you explicitly confirm launch
- `optimization_preference: DEFAULT` unless you override bid strategy

### Algorithm note (May 2026 X source release)

Per the X algorithm breakdown: threads are penalized by author diversity decay (2nd post = 55% of score, 3rd = 32.5%). For organic posts, single tweets outperform threads. For paid: promoted posts are injected directly and bypass feed scoring penalties — threads vs. single post doesn't affect paid performance.
