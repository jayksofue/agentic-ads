# Dry Run Guide

**A "dry run" means seeing exactly what Claude would launch before it costs you anything.** Every platform in this skill supports one — always run a dry run on a new account or a new audience before spending real money.

## How to ask for a dry run

Add any of these to your campaign prompt:

- `"Dry run first"`
- `"Show me what you'd submit before doing anything"`
- `"Validate only — don't launch"`

Claude will build the campaign in a preview state, show you the setup, and wait for your explicit go before anything is created or spent.

## What "dry run" means on each platform

Every platform has a slightly different mechanism for previewing a campaign without spending. Here's what Claude does on each one:

### LinkedIn
Claude builds the campaign in **Draft** status inside Campaign Manager and stops there. Nothing serves, nothing is billed. You review the draft in the LinkedIn UI, and Claude only activates it when you tell it to.

### Meta
Meta's official command-line tool has no "validate only" mode, so the dry-run path is: **build the campaign as PAUSED** (a real object that can't serve or spend), verify it saved that way, and delete it once you're done reviewing. Delete cascades — removing the parent removes the ad sets and ads inside it too. There's an older third-party Meta tool that does have a real "validate only" mode; Claude uses it as a fallback when a request-only preview is genuinely needed.

### X (Twitter)
For the browser path (the default): Claude builds the campaign all the way through Review-and-launch and clicks **Save draft**. The campaign sits as a Draft until you either delete it or explicitly click Publish — never billable in between. For the API path: X's API has no "validate only" flag, so Claude produces a structured summary of everything that would be submitted (campaign, line item, targeting, creative) and waits for your go before making any real call.

### Google
For the browser path (the default): Claude builds the campaign all the way through review, then clicks the ✕ close icon and picks **Discard** in the "Save as a campaign draft?" dialog. Nothing persists — this is the true no-spend dry run. **Do not click Save** — Google Ads has no in-UI way to delete a saved draft once it exists (the only options are Google Ads Editor or publishing-then-deleting, which trips Google's identity re-verification prompt). Just don't save it in the first place. For the API path (only if you already have API access set up): Google's API supports a "validate only" mode that runs all the policy and character-limit checks without creating anything — but it only returns errors on failure; there's no traffic forecast in that response. Forecasts come from a separate service.

### Reddit
Reddit's API has no "validate only" flag either. The dry-run path is: **build the campaign and ad group in the Paused state** (real objects, real IDs, but they don't serve — Claude double-checks by reading the object back to confirm it saved as Paused), and/or use Reddit's separate "Preview an Ad" endpoint to render the creative on its own. Delete once you're done reviewing.

### TikTok
TikTok gives developers a separate **sandbox advertiser account** for testing — that's the cleanest no-spend dry run: full create / read / delete with no real spend, no real review. If you don't have sandbox access, the alternative is to create the campaign in production as **Disabled** with no active ad (nothing serves without an ad), then delete once you're done reviewing. Claude double-checks that the campaign actually saved as Disabled — TikTok has been observed to silently ignore the Disabled flag on some accounts.

## What to check during a dry run

- **Audience size** — Under 10K usually means limited delivery on any platform. Over ~10M means you're probably paying to reach people who don't care.
- **Creative** — Image dimensions match the format? Video aspect ratio right? Copy under the character limit for that platform?
- **Budget pacing** — Is it set as lifetime or daily? Does the start/end date match what you asked for?
- **Placements** — LinkedIn Audience Network off (unless you wanted it on)? Google Display Network off for Search-only campaigns? Meta Audience Network excluded?
- **Targeting conflicts** — Any overlapping exclusions that would zero out the audience?

## Confirming launch after the dry run

Once everything looks right:

```
Looks good — go ahead and launch.
```

Claude activates the campaign on the platform and echoes back the campaign ID + a link to it in Ads Manager.
