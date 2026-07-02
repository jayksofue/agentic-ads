# X (Twitter) Ads — Skill

Claude connects to the X Ads API to create and launch promoted post campaigns.

## Prerequisites

1. **X Ads API access (requires approval)**

   The X Ads API is free once approved but requires a separate application from general X API access. Apply at [ads.x.com/help](https://ads.x.com/help). Approval typically takes weeks and is not guaranteed.

   > **No API access yet?** Claude can still drive the X Ads UI via browser automation — same pattern as the LinkedIn skill. Just say "use browser mode" when starting.

   Once approved, you'll need:
   - A developer app with Ads API (Standard Access) enabled
   - `TWITTER_ADS_ACCOUNT_ID` (find in ads.x.com, in the URL)
   - Consumer key/secret + access token/secret

2. **Set credentials** in your environment or `.env`:

   ```
   TWITTER_CONSUMER_KEY=...
   TWITTER_CONSUMER_SECRET=...
   TWITTER_ACCESS_TOKEN=...
   TWITTER_ACCESS_TOKEN_SECRET=...
   TWITTER_ADS_ACCOUNT_ID=...
   ```

3. **Install the X Ads SDK** (optional, Claude can also call the API directly):

   ```bash
   pip install twitter-ads
   ```

## Campaign setup prompt

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

## Dry run mode

X Ads API doesn't have a native `validate_only` flag, so Claude generates a structured preview:

```
Show me the X campaign payload — don't submit yet.
```

Claude returns:
- Full API request body (campaign + line item + targeting + creative)
- Estimated audience size based on targeting criteria
- Explicit confirmation prompt before any API call is made

## Audience targeting options

- **Keywords**: terms users have tweeted or engaged with
- **Interests**: IAB categories (Finance, Technology, Crypto, etc.)
- **Follower lookalikes**: target users similar to followers of @[handle]
- **Demographics**: age, gender, location, language, device

## Key settings Claude enforces

- `charge_by: ENGAGEMENT` — you pay per engagement, not impression
- Promoted-only tweets: Claude can create a tweet that only appears as an ad (not on your timeline) if you specify
- `optimization_preference: DEFAULT` unless you want to override bid strategy

## Algorithm note (May 2026 X source release)

Per the X algorithm breakdown: threads are penalized by author diversity decay (2nd post = 55% of score, 3rd = 32.5%). For organic posts, single tweets outperform threads. For paid: this doesn't apply — promoted posts are injected directly and bypass feed scoring penalties.
