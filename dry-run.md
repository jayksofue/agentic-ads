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
Claude calls the Marketing API with `execution_options: { validate_only: true }`. The API runs all validation checks (policy, targeting, creative specs) and returns any errors — without creating a campaign or charging anything.

### X (Twitter)
No native validate-only flag. Claude generates the full API payload (campaign + line item + targeting criteria + creative) and shows it to you as structured JSON. No API call is made until you explicitly confirm.

### Google
Claude calls the Google Ads API with `validate_only: true` on all mutate operations. Google runs policy checks, keyword conflicts, ad copy character limits, and targeting validation — without creating anything.

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

Claude activates (LinkedIn), submits (Meta/Google), or calls the create endpoint (X).
