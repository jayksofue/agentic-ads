---
name: agentic-ads-copy
description: Generate and validate ad copy for any ad platform with an anti-slop check before attaching. Enforces platform character limits and preset-CTA rules. Loaded by the parent agentic-ads skill when creative copy is needed.
---

# Ad Copy — Skill

Generates and validates ad copy for any platform. Runs anti-slop before attaching creative.

## When this runs

- **User provides copy**: Claude checks it for slop and flags any issues before proceeding
- **No copy provided**: Claude generates copy using the StoryBrand framework, checks it, and iterates until clean

---

## StoryBrand framework

The customer is the hero. Your product is the guide that helps them win.

| Element | What it is | Example |
|---|---|---|
| **Hook** | The problem your hero is stuck with | "Payments that take 3 days to settle" |
| **Guide** | Your product as the solution | "Eco moves money in seconds, onchain" |
| **CTA** | One clear action | "See how it works" |

Keep the customer's problem in the headline, not your product's features. Features explain what you do; problems explain why they care.

---

## Platform character limits

| Platform | Headline | Body / Intro text | CTA |
|---|---|---|---|
| LinkedIn | 70 chars | 150 chars (intro text) | 44 chars |
| Meta | 40 chars | 125 chars (primary text) | 20 chars |
| X | — | 280 chars total | — |
| Google Search | 30 chars (×3 headlines) | 90 chars (×2 descriptions) | — |

---

## Anti-slop checklist

These patterns weaken copy. Claude checks for all of them and rewrites before proceeding.

| Pattern | Example | Fix |
|---|---|---|
| **Buzzword stacking** | "seamless, innovative, cutting-edge" | Replace with a specific claim |
| **Signposting** | "Here's how it works", "In this post" | Cut it — get to the point |
| **Weak CTA** | "Learn more", "Click here" | Use a specific action: "See a live transfer", "Get the rate" |
| **Feature-first headline** | "Introducing our new API" | Lead with the customer's problem or outcome |
| **Passive voice** | "Payments are processed faster" | "Send payments in seconds" |
| **Vague superlatives** | "the best", "world-class", "leading" | Cut or replace with a verifiable claim |
| **Em dashes as drama** | "Payments — reimagined" | Rewrite without the dash |

If `copy_tells.py` is available at `~/.claude/skills/eco-seo/copy_tells.py`, run it:

```bash
echo "your copy here" | python ~/.claude/skills/eco-seo/copy_tells.py
```

A score of 0 = clean. Any flags = rewrite the flagged line before proceeding.

---

## Copy generation prompt

When generating copy, Claude asks:

1. **What does your product do?** (one sentence)
2. **Who is the audience?** (role, industry, pain point)
3. **What's the one outcome they get?** (specific, measurable if possible)
4. **What's the CTA?** (what should they do next)

Then generates 2–3 variants per platform, checks each for slop, and presents the cleanest option with alternates.

---

## Example output

**Input:** B2B payments product, targeting finance leaders, outcome: faster settlement

**LinkedIn (Document ad)**
> Intro text: Your treasury team is sitting on cash that should already be moving.
> Headline: Settle in seconds, not days
> CTA: See how it works

**Meta (Single image)**
> Primary text: Cross-border payments shouldn't take 3 days. Here's what real-time settlement looks like.
> Headline: Money that moves when you do
> CTA: Watch the demo

**X**
> CFOs are losing yield to slow settlement windows. Real-time treasury starts with the rails underneath. [link]
