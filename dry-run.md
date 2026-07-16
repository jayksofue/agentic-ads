# Dry Run Guide

Every platform in this skill supports a validation step before any campaign goes live or any budget is spent. Always dry run first when launching on a new account or testing a new audience.

## How to request a dry run

Add to any campaign prompt:
- `"Dry run first"` or `"Validate only — don't launch"`
- Or just ask: `"Show me what you'd submit before doing anything"`

## Platform dry run behavior

### LinkedIn
Claude builds the campaign to **Draft** status in Campaign Manager and stops. Nothing is activated, nothing is billed. You review the draft in the UI and tell Claude to activate when ready.

### Meta
The `meta-ads` PyPI CLI has **no `--dry-run` flag**. The QA path is: create the campaign PAUSED (via `meta ads campaign create --status PAUSED` — a real object that cannot serve), verify it by re-reading, then `meta ads campaign delete <ID> --force` (delete cascades to child ad sets and ads). The npm `meta-ads` fallback tool *does* have a `--dry-run` flag (live-verified 2026-07-04 as a true no-op) — use it if you want a request-only preview without creating anything.

### X (Twitter)
Method 1 (browser, live-verified 2026-07-09): build the wizard through Review-and-launch, click **Save draft** — the campaign persists as `Draft` state (never billable) until you delete it or explicitly click Publish. No pre-launch Paused toggle; the mechanism is Save draft. Method 2 (X Ads API v12): no `validate_only` flag; Claude generates the full API payload (campaign + line item + targeting + creative) as structured JSON without submitting until you confirm.

### Google
Method 1 (browser, live-verified): build the wizard, click the ✕ close icon, and choose **Discard** in the "Save as a campaign draft?" modal. Nothing persists — this is the true no-spend dry run. **Do not click Save** — saved drafts have no UI delete control (the audit surfaced this: the trash icon is not exposed for drafts-in-progress, so the only route to remove a saved draft is via Google Ads Editor or by publishing-then-removing, which hits the "Confirm it's you" identity-verification gate). Method 2 (Google Ads API, if you have Basic+ access): `validate_only: true` on mutate operations runs policy/keyword/character/targeting validation — it returns errors only (no forecasts). Weekly-impression forecasts require KeywordPlanIdeaService instead.

### Reddit
No native validate-only flag. Claude builds the campaign and ad group in the paused state (real objects that don't serve — verify by re-reading before any activation), and/or uses the **Preview an Ad** endpoint to render the creative. Delete after QA, same idea as the Meta cycle. Budgets are in micros ($1 = 1,000,000). ⚠️ The paused-state field name is unverified against Reddit's private v3 spec — see [reddit/SKILL.md](./reddit/SKILL.md) safety banner before your first live call.

### TikTok
No native validate-only flag. Claude uses the developer **sandbox** advertiser for a true no-spend dry run, or in production creates the campaign disabled (`operation_status: DISABLE`) with no active ad, then deletes after QA. Note the observed allow-list caveat: some apps report `DISABLE` being silently ignored — Claude re-reads and re-disables the object before proceeding. Budgets are in the account currency as whole units (dollars for USD accounts) — not cents, not micros.

## What to check in a dry run

- **Audience size**: too small (<10k) means limited delivery; too broad means wasted spend
- **Creative specs**: image dimensions, copy length limits, aspect ratios
- **Budget pacing**: lifetime vs daily, start/end dates
- **Placement exclusions**: confirm Audience Network / Display Network is off if you didn't intend it
- **Targeting conflicts**: overlapping exclusions that zero out the audience

## Confirming launch after dry run

Once you've reviewed:
```
Looks good — go ahead and launch.
```

Claude activates (LinkedIn), submits (Meta/Google), or calls the create endpoint (X/Reddit/TikTok).
