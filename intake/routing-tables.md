# Routing Tables — verified 2026-07-12

## CAUTION LIST — verifier flagged (eyeball before ship)

| Platform | Claim | Verdict | What's actually true |
|---|---|---|---|
| LinkedIn | `objectiveType` uses plural forms (`WEBSITE_VISITS`, `VIDEO_VIEWS`, `WEBSITE_CONVERSIONS`, `JOB_APPLICANTS`) | REFUTED | Every JSON example in LinkedIn's own docs + the objective-mapping doc uses SINGULAR (`WEBSITE_VISIT`, `VIDEO_VIEW`, `WEBSITE_CONVERSION`, `JOB_APPLICANT`). Only the schema-table enum uses plural — appears to be a doc-writing error. Default to singular; verify via GET on a live campaign before hardcoding. |
| LinkedIn | `WEBSITE_TRAFFIC` for Dynamic Ads | ADD | Appears in official Dynamic Ad JSON examples but is not in either enum listing. |
| LinkedIn | `min_daily_usd: 10`, `min_lifetime_usd: 100`, `practical_min_daily_usd: 25`, sweet spot 50K–400K | UNVERIFIABLE | Not in LinkedIn primary docs. Agency-benchmark heuristics only. |
| Meta | THRUPLAY only counts 15s watches (rule 10) | REFUTED | THRUPLAY counts 15s OR full video, whichever comes first — a 5s creative DOES register on full-play. Recommendation to use `TWO_SECOND_CONTINUOUS_VIDEO_VIEWS` for short creatives is fine; the reason given is wrong. |
| Meta | Feed video `duration_seconds_max: 241` | REFUTED | Meta's cap is 240 **minutes** / 4 GB. Likely mis-transcription of "up to 241 minutes." |
| Meta | Reels max 90s | STALE | Now up to 180s on Instagram. |
| Meta | Absolute daily floors ($1 impressions high-cost / $2.50 clicks / $5 high-cost / $40 low-frequency) | UNVERIFIABLE | Not doc-sourced; third-party guides disagree. Treat as heuristics. |
| X | Objective enum omits `FOLLOWERS` and `PREROLL_VIEWS` | INCOMPLETE | Both are valid values per docs.x.com. Route user goal "followers" natively to `FOLLOWERS`. |
| X | `DRAFT` not in `entity_status` enum | REFUTED | `DRAFT` IS a valid `entity_status` value alongside `ACTIVE`/`PAUSED` per docs.x.com. |
| X | "Optimized Targeting default ON" + API field `audience_expansion` | REFUTED / UNVERIFIABLE | Audience Expansion is opt-in via checkbox (Defined/Expanded/Broad), not default-ON, and NOT supported on App installs or Website traffic. Exact API field name is not primary-sourced. Researcher conflated Audience Expansion with Optimized Targeting. |
| Google Ads | "$1/day official minimum" | UNVERIFIABLE | Not in Google Ads API primary docs (`CampaignBudget` reference). Help-center folklore only. |
| Google Ads | Default `target_search_network=true` when omitted | UNVERIFIABLE | Undocumented. Always set `network_settings` explicitly. |
| Reddit | "Crypto not available on self-serve; sales rep required" | PARTIALLY REFUTED | Overstated. Crypto is *restricted* (needs licensing + Reddit review, may route through rep in practice), not banned from self-serve. Crypto **loans** are banned outright. |
| Reddit | Objective enum spellings (`BRAND_AWARENESS`, `TRAFFIC`, etc.) | UNVERIFIABLE | Reddit v3 API docs are behind an allow-list. Field name `objective` confirmed via Fivetran schema; exact SNAKE_CASE values are not primary-sourced. |
| Reddit | `adsedit` scope requires partner allow-list in 2026 | UNVERIFIABLE | Multiple secondary integrators agree, but Reddit's own page is unreachable. |
| TikTok | `auto_targeting_enabled` is the cold-audience-expansion flag, default ON | REFUTED | Tagged "To be deprecated" since June 2024. Current fields are `smart_audience_enabled` and `smart_interest_behavior_enabled` (two separate booleans, both opt-in, default false/unset). |
| TikTok | Bid enums (`BID_TYPE_NO_BID`, `BID_TYPE_CUSTOM`) at campaign level | STALE | v1.3 deprecated `bid_type`, `deep_bid_type`, `roas_bid`, `optimize_goal` at the campaign level. Bidding fields now live on the ad group. |
| TikTok | Budget floors ($50 campaign / $20 ad group) | HIGH-CONFIDENCE, NOT PRIMARY-VERIFIED | Field is `budget` (float, account currency, not cents/micros) — confirmed. Specific floor numbers come from the currency-verification-ratio table which was not re-fetched. |

---

## LinkedIn

### Goal → Objective enum
API field: `objectiveType` on the Campaign entity. **Use SINGULAR forms** (see CAUTION).

| User goal | Enum | Notes |
|---|---|---|
| traffic | `WEBSITE_VISIT` | Also `WEBSITE_TRAFFIC` in Dynamic Ad examples |
| awareness | `BRAND_AWARENESS` | MAX_REACH / MAX_IMPRESSION |
| engagement | `ENGAGEMENT` | Also for Follower ads (`FOLLOW_COMPANY`) |
| video_views | `VIDEO_VIEW` | SINGLE_VIDEO format only; `costType=CPV`, `MAX_VIDEO_VIEW` |
| conversions | `WEBSITE_CONVERSION` | Requires Insight Tag conversion associated |
| leads | `LEAD_GENERATION` | `offsiteDeliveryEnabled` MUST be `false` |
| app_installs | NOT SUPPORTED | LinkedIn has no APP_INSTALL. Route elsewhere or map to `WEBSITE_VISIT` with warning |
| sales | `WEBSITE_CONVERSION` | Configure purchase event via Insight Tag |
| talent | `TALENT_LEAD` | Present in mapping doc, absent from schema enum — Talent Solutions accounts |

Source: https://learn.microsoft.com/en-us/linkedin/marketing/integrations/ads/account-structure/campaign-objectives?view=li-lms-2026-07 | https://learn.microsoft.com/en-us/linkedin/marketing/integrations/ads/account-structure/create-and-manage-campaigns?view=li-lms-2026-07

### Budget

| Field | Value |
|---|---|
| API field | `dailyBudget.amount` / `totalBudget.amount` (with `.currencyCode`) |
| Unit | BigDecimal, whole-currency string (e.g. `"18"` or `"3000.00"`) — NOT cents or micros |
| Hard min USD (daily) | $10 (agency-cited, not in primary docs) |
| Practical min USD | $25/day Sponsored Content, $10/day Text Ads |
| Delivery ceiling | LinkedIn may charge up to 150% of daily budget on high-opportunity days (CONFIRMED verbatim) |

Source: https://learn.microsoft.com/en-us/linkedin/marketing/integrations/ads/account-structure/create-and-manage-campaigns?view=li-lms-2026-07

### Cold-audience-expansion flag

| Field | Value |
|---|---|
| UI name | Audience Expansion |
| API field | `audienceExpansionEnabled` |
| Off value | `false` (default) |

Related: `offsiteDeliveryEnabled` (LAN toggle) — no server default, required on create. MUST be `false` for `LEAD_GENERATION` and Message/Conversation ads.

### Paused-state field

| Field | Value |
|---|---|
| API field | `status` |
| Paused value | `PAUSED` (or `DRAFT` for pre-launch) |
| Active value | `ACTIVE` |
| Full enum | `ACTIVE, PAUSED, ARCHIVED, COMPLETED, CANCELED, DRAFT, PENDING_DELETION, REMOVED` |

### Regulated categories

| Field | Value |
|---|---|
| Crypto allowed? | Yes, restricted — pre-authorization required via LinkedIn crypto ads team |
| Geo | US, UK, CH, DE, FR, NL, UAE only |
| Targeting | Finance professionals only; exclude under-18 |
| Prohibited (even with auth) | Specific coins/tokens, ICOs, DAOs, DeFi, mining, staking, NFTs, guaranteed returns |
| Stablecoins | Not named — treat as restricted under crypto umbrella; confirm with LinkedIn BD |
| Political declaration (EU) | `politicalIntent` = `POLITICAL` \| `NOT_POLITICAL` \| `NOT_DECLARED` required |
| Declaration API field | None — vetting is out-of-band via account approval |

Source: https://www.linkedin.com/help/lms/answer/a9401025

### Audience-size sweet spot
50,000 – 400,000 members (heuristic, not primary-sourced). `CAMPAIGN_AUDIENCE_COUNT_HOLD` serving status kicks in below LinkedIn's undisclosed threshold.

### Creative specs

| Format | Aspect | Resolution | Max size | Character limits |
|---|---|---|---|---|
| Sponsored Content — Single Image | 1.91:1 / 1:1 / 4:5 | 1200x628 / 1200x1200 / 720x900 | 5 MB (JPG/PNG/GIF) | Headline 70 / Intro 150 (600 max) |
| Sponsored Content — Carousel | 1:1 | 1080x1080/card | 10 MB | Intro 255 / Card headline 45; 2–10 cards; not on LAN |
| Sponsored Content — Video | 16:9 / 1:1 / 4:5 | 1080p (360p min) | 200 MB (MP4) | Intro 600 / Headline 70; 3s–30min (15–30s best) |
| Text Ads | 1:1 | 100x100 | 2 MB | Headline 25 / Description 75; desktop only |
| Message / Conversation | n/a | Optional 300x250 banner | 2 MB | Subject 60 / Body 1500 / CTA 20; billed CPS; not on LAN |

### Bid strategy defaults

| Context | API |
|---|---|
| Cold, first 2 weeks | Automated (Maximum Delivery). `optimizationTargetType` = `MAX_IMPRESSION` / `MAX_CLICK` / `MAX_LEAD` / `MAX_CONVERSION` |
| Post-learning, known CPA | `CAP_COST_AND_MAXIMIZE_CLICKS` / `_LEADS`, `TARGET_COST_PER_CLICK` / `_IMPRESSION` |
| Manual bidding | `optimizationTargetType=NONE`, `costType=CPC` \| `CPM` \| `CPV` |

### Recommendation rules

1. If daily budget < $25 on Sponsored Content → warn: $10 platform min but $25–100 needed for signal; CPCs run $5–12.
2. If first-time advertiser → set `audienceExpansionEnabled=false` and `offsiteDeliveryEnabled=false` explicitly.
3. If goal is conversions/leads AND audience < 50K → block; broaden to 50K–400K (avoids `CAMPAIGN_AUDIENCE_COUNT_HOLD`).
4. If `MAX_CONVERSION` AND no Insight Tag conversion associated → refuse activation; API rejects.
5. If regulated vertical (crypto/stablecoin/fintech) → require BD pre-authorization + geo allowlist + finance-professional targeting + under-18 exclusion before spending.
6. If any EU country in targeting → require `politicalIntent` declaration.
7. If user asks for "app installs" → don't silently map; route to Meta/TikTok/Google UAC, or explicitly to `WEBSITE_VISIT` with a store page and a warning.
8. Always create as `status=DRAFT`, GET-verify, then flip to `ACTIVE`.
9. Keep `creativeSelection=OPTIMIZED` (default); use `ROUND_ROBIN` only for Message/Conversation ads on non-lead objectives.
10. Sponsored Message/Conversation ads: `offsiteDeliveryEnabled=false`; billed CPS (CPM × 1000).

### Unverified / speculative
See CAUTION LIST rows for LinkedIn.

---

## Meta (Facebook / Instagram)

### Goal → Objective enum
API field: `objective` on campaign entity.

| User goal | Enum |
|---|---|
| traffic | `OUTCOME_TRAFFIC` |
| awareness | `OUTCOME_AWARENESS` |
| engagement | `OUTCOME_ENGAGEMENT` |
| video_views | `OUTCOME_ENGAGEMENT` (+ optimization_goal `THRUPLAY` or `TWO_SECOND_CONTINUOUS_VIDEO_VIEWS`) |
| conversions | `OUTCOME_SALES` (or `OUTCOME_LEADS` if non-sale intent) |
| leads | `OUTCOME_LEADS` |
| app_installs | `OUTCOME_APP_PROMOTION` |
| sales | `OUTCOME_SALES` |

Source: https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group

**Requires pixel/CAPI:** `OUTCOME_SALES`, `OUTCOME_LEADS` (pixel event variant).

### Budget

| Field | Value |
|---|---|
| API field | `daily_budget` / `lifetime_budget` |
| Unit | **cents** (`daily_budget=2000` → $20) |
| Hard min USD | Not doc-verified — see CAUTION |
| Practical min USD | ~$20/day (needs ~50 events/week to exit learning) |

Source: https://developers.facebook.com/docs/marketing-api/bidding/overview/budgets

### Cold-audience-expansion flag

| Field | Value |
|---|---|
| UI name | Advantage+ Audience |
| API field | `targeting.targeting_automation.advantage_audience` |
| Off value | `0` (default ON) |

Meta is progressively forcing this ON for Sales/Leads in some markets — `= 0` may be silently ignored on Advantage+ Shopping.

### Paused-state field

| Field | Value |
|---|---|
| API field | `status` |
| Paused value | `PAUSED` |
| Active value | `ACTIVE` |
| Create-time values | Only `ACTIVE` or `PAUSED` accepted |

`effective_status` is a read-only expanded field — do not confuse with writable `status`.

### Regulated categories

| Field | Value |
|---|---|
| Declaration API field | `special_ad_categories` (enum: `CREDIT`, `HOUSING`, `EMPLOYMENT`, `ISSUES_ELECTIONS_POLITICS`, `ONLINE_GAMBLING_AND_GAMING`, `FINANCIAL_PRODUCTS_SERVICES` [India]) |
| Crypto allowed? | Yes with **written permission via Business Suite → Authorizations & Verifications** — separate from `special_ad_categories` |
| Crypto scope requiring permission | Exchanges, borrow/lend, enhanced wallets, mining software, investment solicitation |
| Not requiring permission | Storage-only wallets, blockchain B2B services, mining hardware, tax services, education without product offer |
| For Eco (stablecoin B2B infra) | Generally outside written-permission scope IF creative avoids consumer investment/yield/staking language. Any consumer-facing US ad mentioning buying/holding/earning yield triggers the requirement. |

Source: https://transparency.meta.com/policies/ad-standards/restricted-goods-services/cryptocurrency-products-and-services/

### Audience-size sweet spot
1M–10M MAU (industry heuristic; Meta doesn't publish).

### Creative specs

| Format | Aspect | Resolution | Duration | Max size | Character limits |
|---|---|---|---|---|---|
| Feed image | 1:1 (rec) / 1.91:1–4:5 | 1080x1080 min | — | 30 MB | Text 125 / Headline 40 / Desc 30 |
| Stories/Reels image | 9:16 | 1080x1920 | — | 30 MB | Top/bottom 250px safe zone |
| Feed video | 1:1 or 4:5 | 1080p+ | See CAUTION | 4 GB | Text 125 |
| Reels video | 9:16 | 1080x1920 | See CAUTION (stale) | 4 GB | — |
| Stories video | 9:16 | 1080x1920 | 60s | 4 GB | — |
| Carousel | 1:1 | 1080x1080/card | — | 30 MB | Headline 40/card, desc 20; 2–10 cards |

### Bid strategy defaults

| Context | API |
|---|---|
| Cold, no CPA history | `LOWEST_COST_WITHOUT_CAP` |
| Known CPA, want ceiling | `COST_CAP` (~20% above current CPA) |
| eCom + catalog | `LOWEST_COST_WITH_MIN_ROAS` |
| Strict bid ceiling | `LOWEST_COST_WITH_BID_CAP` |

### Recommendation rules

1. If conversions/leads AND no pixel/CAPI → block; don't silently fall back to `OUTCOME_TRAFFIC`.
2. If cold audience → `advantage_audience=0` for first 2 weeks; `LOWEST_COST_WITHOUT_CAP`; 1–3 interests only.
3. If `daily_budget < 5× expected CPA` → warn: won't exit learning; consolidate ad sets or raise budget.
4. If `special_ad_categories` set → auto-disable detailed interests, age narrowing, gender, ZIP radius <15mi; lookalike ≥1–15%.
5. If crypto exchange / DeFi / staking-wallet / investment-solicitation → require written permission before create.
6. If stablecoin B2B (Eco) → `special_ad_categories=[]` BUT flag creative mentioning consumer yield/staking/holding — triggers crypto permission even for infra brands.
7. Always create as `status=PAUSED`, human-review in Ads Manager, then flip `ACTIVE`.
8. Set `publisher_platforms` explicitly; omit `audience_network` unless intentional.
9. If audience < 500K prospecting → widen; algo needs 1M–10M for stable learning.
10. Short video (<15s) → use `TWO_SECOND_CONTINUOUS_VIDEO_VIEWS`, not `THRUPLAY` (but see CAUTION on THRUPLAY mechanics).

### Unverified / speculative
See CAUTION LIST rows for Meta.

---

## X (formerly Twitter)

### Goal → Objective enum
API field: `objective` on line_item.

| User goal | Enum | Notes |
|---|---|---|
| awareness | `REACH` | Cheapest CPM |
| engagement | `ENGAGEMENTS` | |
| traffic | `WEBSITE_CLICKS` | |
| conversions | `WEBSITE_CONVERSIONS` | Requires X Pixel or CAPI |
| sales | `WEBSITE_CONVERSIONS` | Pair with Purchase pixel event; DPA supported |
| video_views | `VIDEO_VIEWS` | |
| video_views (premium pre-roll) | `PREROLL_VIEWS` | (added per verifier) |
| app_installs | `APP_INSTALLS` | |
| app_engagement | `APP_ENGAGEMENTS` | |
| followers | `FOLLOWERS` | Native — do NOT route elsewhere |
| leads | `WEBSITE_CONVERSIONS` (workaround) | X has no native LEADS objective; native Lead Gen Cards deprecated |

Source: https://docs.x.com/x-ads-api/campaign-management/reference

**Requires pixel:** `WEBSITE_CONVERSIONS`.

### Budget

| Field | Value |
|---|---|
| API field | `daily_budget_amount_local_micro` / `total_budget_amount_local_micro` |
| Unit | micros (1,000,000 = $1) |
| Hard min | X states: "no minimum spend required" |
| Practical min USD | ~$5–10/day (third-party heuristic); $50/day for conversion objectives |

Source: https://business.x.com/en/help/overview/ads-pricing

### Cold-audience-expansion flag

| Field | Value |
|---|---|
| UI name | Audience Expansion (opt-in checkbox: Defined / Expanded / Broad) — NOT default ON |
| Not supported on | App installs, Website traffic (those use Automated Targeting) |
| API field | Not primary-sourced; verify via live GET |

Related but distinct: Optimized Targeting (only disable-able in Advanced flow) — API field name not primary-sourced.

### Paused-state field

| Field | Value |
|---|---|
| API field | `entity_status` |
| Values | `ACTIVE`, `PAUSED`, `DRAFT` (all valid), `DELETED` |
| Paused value | `PAUSED` |
| Active value | `ACTIVE` |

Source: https://docs.x.com/x-ads-api/campaign-management/reference

### Regulated categories

| Field | Value |
|---|---|
| Crypto allowed? | Yes with prior certification via business.x.com Financial Services / Cryptocurrency form + country license |
| US requirement | SEC / CFTC / FinCEN registration |
| Prohibited (even with cert) | ICOs, IEOs, IDExOs, mining hardware/software |
| Permitted with cert | Exchanges, wallets, kiosks/ATMs, crypto credit/debit cards, staking, crypto CFDs, tax calculators, DeFi lending/borrowing, DApps, DEXes |
| Permitted without cert | Smart contracts + educational content on blockchain/crypto/DeFi |
| Not available for crypto | Belgium, Greece, Qatar, Russia, Singapore, Slovenia, Ukraine |
| Declaration API field | None — certification tied to ads account, not campaign object |

Source: https://business.x.com/en/help/ads-policies/ads-content-policies/financial-services

### Audience-size sweet spot
1M–50M (heuristic; X publishes no numeric band).

### Creative specs

| Format | Aspect | Resolution | Duration | Max size | Character limits |
|---|---|---|---|---|---|
| Text Ad | — | — | — | — | Post 280 (257 with URL) |
| Image Ad | 1:1 / 1.91:1 / 4:5 / 2:3 / 16:9 / 9:16 | 1200x1200 rec | — | 5 MB (PNG/JPEG) | Post 280 / Website Card headline 70 |
| Video Ad | 1:1 / 16:9 (rec) | 1080p | 140s (10min select) | 1 GB (MP4/MOV) | ≤15s best, <60s loops |
| Vertical Video (Immersive) | 9:16 rec | 1080x1920 | 140s | — | Bitrate 5–10 Mbps |
| Carousel | Single ratio per carousel | 800x800 min | — | — | Title 70/card; 2–6 slides |
| Amplify Pre-roll | 1:1 (rec) | 1200x1200 | 140s (≤15s rec) | 1 GB | — |

### Bid strategy defaults

| Context | API |
|---|---|
| Cold, no history | `bid_strategy=AUTO` (no `bid_amount_local_micro`) |
| 50+ conv/week | `bid_strategy=TARGET` + `bid_amount_local_micro` |
| Max delivery, mature | `bid_strategy=MAX` |

### Recommendation rules

1. Cold TOF prospecting → default `REACH`, not `WEBSITE_CLICKS` (X auction is smaller than Meta/Google; cheap CPM seeds retargeting pool).
2. Any conversion/app-install objective with no history → `bid_strategy=AUTO`.
3. Crypto/stablecoin advertiser (Eco) → verify X Cryptocurrency/DeFi certification via business.x.com AND country registration BEFORE any create; ICOs/IEOs/IDExOs and mining forbidden even with cert.
4. Educational content only, no product CTA → advertise without financial services cert (X explicitly permits blockchain/crypto/DeFi education).
5. Programmatic creation → `entity_status=PAUSED`, verify, then `ACTIVE`.
6. Custom Audience upload → ≥500 matched users minimum; below ~10K delivery is thin — prefer Follower LAL / Interest / Keyword / Conversation Topic.
7. If budget <$50/day AND conversion/app-install objective → warn; default to Engagements or Reach first 2 weeks.
8. Advanced flow cold targeting → explicitly disable Optimized Targeting (Simple flow can't).
9. Default to 1:1 or 9:16 vertical, <15s video, image ≤5 MB, copy ≤100 chars.
10. `WEBSITE_CONVERSIONS` or DPA → confirm X Pixel/CAPI firing Page View, Content View, Add to Cart, Purchase before create.

### Unverified / speculative
See CAUTION LIST rows for X. Also: `twitter-python-ads-sdk` last released v11.0.0 on 2022-05-27 — functionally abandoned though not officially deprecated.

---

## Google Ads

### Goal → Objective enum
API field: `campaign.advertising_channel_type`.

| User goal | Enum | Notes |
|---|---|---|
| traffic | `SEARCH` | Pair with `TARGET_SPEND` or `MANUAL_CPC` |
| awareness | `DEMAND_GEN` | Or `DISPLAY` (legacy). Discovery replaced by Demand Gen |
| engagement | `DEMAND_GEN` | YouTube/Discover/Gmail feed |
| video_views | `VIDEO` | `TARGET_CPV`. For conversion-focused video, use `DEMAND_GEN` — Video Action Campaigns fully sunset (all migrated by April 2026) |
| conversions | `SEARCH` | `MAXIMIZE_CONVERSIONS` / `TARGET_CPA` |
| leads | `SEARCH` | Same; lead-form assets |
| app_installs | `MULTI_CHANNEL` | `advertising_channel_sub_type=APP_CAMPAIGN` |
| sales | `PERFORMANCE_MAX` | `MAXIMIZE_CONVERSION_VALUE` + Target ROAS |

Full v23 enum: `DEMAND_GEN, DISPLAY, HOTEL, LOCAL, LOCAL_SERVICES, MULTI_CHANNEL, PERFORMANCE_MAX, SEARCH, SHOPPING, SMART, TRAVEL, VIDEO` (CONFIRMED).

Source: https://developers.google.com/google-ads/api/reference/rpc/v23/AdvertisingChannelTypeEnum.AdvertisingChannelType

**Requires conversion tracking:** conversions, leads, sales.

### Budget

| Field | Value |
|---|---|
| API field | `campaign_budget.amount_micros` |
| Unit | micros (`int64`; 1,000,000 = 1 currency unit) — CONFIRMED |
| Monthly cap | `30.4 × amount_micros/1e6` — CONFIRMED verbatim |
| Daily vs lifetime | `amount_micros` when `period=DAILY` (default); `total_amount_micros` only when `period=CUSTOM_PERIOD` |
| Hard min | Not in API primary docs (see CAUTION) |
| Practical min | 10× expected CPC (~$10–50/day per Google beginner guidance) |
| Sharing | `explicitly_shared=true` for shared budgets (default `true`) |

Source: https://developers.google.com/google-ads/api/reference/rpc/v23/CampaignBudget

### Cold-audience-expansion flag (Search)

| Field | Value |
|---|---|
| UI name | Google Search Partners |
| API field | `campaign.network_settings.target_search_network` |
| Off value | `false` |

`target_partner_search_network` is a DIFFERENT field — doc verbatim: "This does not control whether ads will be served on Google Search Partners Network; use `target_search_network` for that instead."

For Demand Gen / Display / PMax the equivalent flag is `optimized_targeting` or `audience_expansion_level`.

Source: https://developers.google.com/google-ads/api/reference/rpc/v23/Campaign.NetworkSettings

### Paused-state field

| Field | Value |
|---|---|
| API field | `campaign.status` |
| Paused value | `PAUSED` |
| Active value | `ENABLED` |
| Full enum | `ENABLED, PAUSED, REMOVED, UNKNOWN, UNSPECIFIED` (no `DRAFT`) |

Source: https://developers.google.com/google-ads/api/reference/rpc/v23/CampaignStatusEnum.CampaignStatus

### Regulated categories

| Field | Value |
|---|---|
| Certification | "Cryptocurrencies and related products" — apply inside Google Ads (Admin → Policy → Account → Apply for certification) |
| Permitted with cert | (1) Exchanges (2) Software wallets (3) Hardware wallets (4) Coin trusts (US only) |
| Prohibited (no cert available) | ICOs, DeFi trading protocols, buying/selling/trading crypto, crypto loans, IDOs, liquidity pools, unhosted software wallets, unregulated dApps, aggregator/comparison sites, trading signals, investment advice, NFT gambling |
| For Eco (stablecoin infra) | Position strictly B2B — no messaging about acquiring/holding/trading/yield on stablecoins |
| Approved locations | ~40 countries: US (FinCEN MSB + state MTL), EU (MiCA CASP since Apr 23, 2025), UK (FCA), CH (FINMA), JP (FSA), UAE (VARA/FSRA), etc. Non-listed = ineligible. |
| Declaration API field | `customer.financial_services_verification_status` (applied via UI, not campaign-level) |

Source: https://support.google.com/adspolicy/answer/14009787

### Audience-size sweet spot
N/A for keyword-based Search targeting (Google doesn't expose an equivalent to Meta/LinkedIn audience estimator). Reach forecasts exist for Display/Video/Demand Gen via `ReachPlanService`.

### Creative specs

| Format | Aspect | Resolution | Max size | Character limits |
|---|---|---|---|---|
| Responsive Search Ad | — | — | — | 15 × 30-char headlines; 4 × 90-char descriptions; 2 × 15-char paths (3H/2D min) |
| Search image assets | 1:1 + 1.91:1 | 300x300 min; 1200x1200 / 1200x628 rec | 5 MB | — |
| Demand Gen single-image | 1:1 / 1.91:1 / 4:5 | 1200x1200 / 1200x628 / 960x1200 | 5 MB | Headline 40 / long 90 / desc 90 / business 25 |
| Demand Gen video | 16:9 / 1:1 / 9:16 | — | YouTube-hosted | up to 180s | — |
| Responsive Display Ad | 1.91:1 + 1:1 imgs; 4:1 + 1:1 logos | 1200x628 / 1200x1200 / 1200x300 | 5 MB | Short 30 / long 90 / desc 90 |
| Performance Max asset group | 1.91:1/1:1/4:5 imgs; 16:9/1:1/9:16 video | Same as above | 5 MB | 3H/2LongH/2D/1business/1logo/1landscape/1square min |

### Bid strategy defaults

| Context | API |
|---|---|
| New Search, no history, traffic | `TARGET_SPEND` (Maximize Clicks) |
| Awareness / branded | `MANUAL_CPC` w/ Enhanced CPC |
| 15+ conv in 30d, leads | `MAXIMIZE_CONVERSIONS` |
| Revenue + Merchant | `MAXIMIZE_CONVERSION_VALUE` |
| Top-of-page brand | `TARGET_IMPRESSION_SHARE` |
| Demand Gen video | `TARGET_CPV` or `MAXIMIZE_CONVERSIONS` |
| Performance Max | `MAXIMIZE_CONVERSIONS` default |

### Recommendation rules

1. New Search account, no conv history → `TARGET_SPEND` or `MANUAL_CPC`, NOT `TARGET_CPA`/`MAXIMIZE_CONVERSIONS` (need 15–30 conv/30d).
2. Stablecoin / DeFi / crypto exchange / software wallet → restrict to B2B/infra/payment-acceptance/education messaging. Prohibited (even with cert): ICOs, DeFi trading, buy/sell/trade, loans, unhosted wallets, aggregators.
3. New Search campaign cold start → explicitly `network_settings.target_content_network=false`, consider `target_search_network=false`, leave `target_google_search=true`.
4. Any create → `status=PAUSED` + `validate_only=true` first mutate.
5. `sales`/`conversions` AND no pixel → block or bridge with `MAXIMIZE_CLICKS`.
6. User asks "Video Action Campaign" or "YouTube conversion campaign" → route to `DEMAND_GEN` (Video Action fully sunset).
7. Test-level dev token → warn: only usable against TEST manager accounts. Basic Access requires manual application (~5 business days).
8. If daily budget < 10× expected CPC → recommend raising or narrowing keywords.
9. `PERFORMANCE_MAX` → NetworkSettings is not honored; do not try to control placement.
10. `SEARCH` cold start → `target_search_network=false` AND `target_content_network=false`; keep `target_google_search=true`.

### Unverified / speculative
See CAUTION LIST rows for Google Ads.

---

## Reddit

### Goal → Objective enum
API field: `objective`. **Enum spellings are PLAUSIBLE, not primary-sourced** — verify via live GET before hardcoding.

| User goal | Enum | Notes |
|---|---|---|
| awareness | `BRAND_AWARENESS` | Merged w/ Reach in UI |
| traffic | `TRAFFIC` | |
| engagement | `TRAFFIC` (workaround) | No dedicated Engagement objective |
| video_views | `VIDEO_VIEWS` | |
| conversions | `CONVERSIONS` | Requires Reddit Pixel |
| app_installs | `APP_INSTALLS` | MMP integration |
| leads | `LEAD_GENERATION` | Beta as of 2026 |
| sales | `CATALOG_SALES` | Beta; catalog upload |

Source: https://business.reddithelp.com/s/article/Ad-campaign-objectives (page returns CSS-stub via WebFetch; schema field name via https://github.com/fivetran/dbt_reddit_ads_source/blob/main/models/docs.md)

**Requires pixel:** `CONVERSIONS`, `CATALOG_SALES`.

### Budget

| Field | Value |
|---|---|
| API field | `goal_value` (on **ad group**, not campaign) |
| Related field | `goal_type` = `DAILY_SPEND` \| `LIFETIME_SPEND` |
| Unit | microcurrency (dollars × 1,000,000) |
| Hard min USD (daily) | $5 (industry-cited) |
| Hard min USD (lifetime) | $25 |
| Practical min USD | $50–100/day |

Full `goal_type` enum: `IMPRESSIONS, PERCENTAGE, CLICKS, CONVERSIONS, LIFETIME_SPEND, DAILY_SPEND, VIDEO_VIEWABLE_IMPRESSIONS` (CONFIRMED from Fivetran schema).

Source: https://github.com/fivetran/dbt_reddit_ads_source/blob/main/models/docs.md

### Cold-audience-expansion flag

| Field | Value |
|---|---|
| UI name | Expand Targeting |
| API field | `expand_targeting` (boolean) — CONFIRMED |
| Off value | `false` (default) |

### Paused-state field

| Field | Value |
|---|---|
| API field | `configured_status` |
| Paused value | `PAUSED` |
| Active value | `ACTIVE` |
| Full enum | `ACTIVE, PAUSED, ARCHIVED, DELETED` |

`effective_status` is a separate read-only derived field.

### Regulated categories

| Field | Value |
|---|---|
| Crypto allowed? | Restricted, not banned. Requires licensing (US: FinCEN registration or documented exemption) + Reddit review. Loans banned outright. |
| Self-serve | Often routed through a Reddit rep in practice; not a documented hard block |
| Prohibited outright | Crypto loans, single-token/coin promotion, ICOs/IDOs, penny stocks, mixers/tumblers, DAOs w/o centralized entity, "50x/100x" claims |
| Declaration API field | None public — vetting out-of-band |

Source: https://business.reddithelp.com/s/article/financial-cryptocurrency-products-and-services-policy

### Audience-size sweet spot
50K–5M members (Reddit recommends ≥50K–100K for consistent delivery).

### Creative specs

| Format | Aspect | Resolution | Duration | Max size | Character limits |
|---|---|---|---|---|---|
| Image | 1:1 / 4:5 / 4:3 / 16:9 | 1080x1080 rec | — | 3 MB (JPG/PNG) | Headline 100 safe / 150 feed max |
| Video | 1:1 / 4:5 / 16:9 | 1080p rec | 2s–15 min (5–30s best) | 1 GB (MP4/MOV) | Headline 100/150 |
| Carousel | 1:1 / 4:5 (shared) | 1080x1080/1080x1350 | — | 3 MB/card | 2–6 cards |

### Bid strategy defaults

| Context | API |
|---|---|
| Cold, first 2 weeks | `MAXIMIZE_VOLUME` |
| Warm, 500+ clicks, proven CPA | `MANUAL_BIDDING` |
| Reserved/premium (Takeovers, First View) | `BIDLESS` (sales-booked only) |

### Recommendation rules

1. Crypto/stablecoin/DeFi (Eco) → note licensing (FinCEN reg or exemption) required; crypto loans banned; may route through rep.
2. Cold audience create → `expand_targeting=false`; start with subreddit targeting, not interest categories.
3. Daily budget <$50 → warn: technical min $5 but need $50–100/day for algo to learn.
4. Conversions / App Installs → require Reddit Pixel or MMP firing events before create; `goal_type=CONVERSIONS` and `optimization_strategy_type=DOWNSTREAM_CONVERSIONS` need signal.
5. Community targeting <50K members → flag delivery risk but don't block; niche subs can convert well.
6. First API campaign on account → confirm allow-list for `adsedit` scope (Conversions API scope alone insufficient) — POST/PATCH returns 403 without approval.
7. Programmatic create → `configured_status=PAUSED`, GET-verify, then flip `ACTIVE` in a separate call. There is no `is_enabled` field.
8. Budget setting → set on **ad group** via `goal_type`/`goal_value` in micros, NOT on campaign.
9. Cold audience, no data → default `bid_strategy=MAXIMIZE_VOLUME`. Switch to `MANUAL_BIDDING` only after >500 clicks + proven CPA.
10. Headlines >100 chars → warn: truncates in conversation placements; 100-char safe universal.

### Unverified / speculative
See CAUTION LIST rows for Reddit.

---

## TikTok

### Goal → Objective enum
API field: `objective_type`. All values CONFIRMED against v1.3 primary docs.

| User goal | Enum | Notes |
|---|---|---|
| traffic | `TRAFFIC` | |
| awareness | `REACH` | |
| awareness (guaranteed) | `RF_REACH` | Reach & Frequency; allowlist; `/adgroup/rf/create/` endpoint |
| engagement | `ENGAGEMENT` | UI: "Community interaction" |
| video_views | `VIDEO_VIEWS` | |
| conversions | `WEB_CONVERSIONS` | Requires Pixel or Events API |
| leads | `LEAD_GENERATION` | Instant Form / Website / DMs / IM / phone |
| app_installs | `APP_PROMOTION` | `app_promotion_type` = `APP_INSTALL` \| `APP_RETARGETING` \| `APP_PREREGISTRATION`; iOS uses `campaign_type=IOS14_CAMPAIGN` |
| sales | `PRODUCT_SALES` | `campaign_product_source` = `CATALOG` \| `STORE`; being merged w/ Web Conversions into virtual "Sales" via `virtual_objective_type` + `sales_destination` |

Source: https://business-api.tiktok.com/portal/docs?id=1739318962329602

**Requires pixel/events:** `WEB_CONVERSIONS`, `PRODUCT_SALES` (CATALOG variant needs event source; STORE/TikTok Shop doesn't require advertiser's own pixel).

### Budget

| Field | Value |
|---|---|
| API field | `budget` |
| Unit | **float in account currency** — NOT cents, NOT micros |
| Precision (USD) | 0.01 |
| Hard min USD (daily) | $50 campaign / $20 ad group (per currency-verification-ratio table; not re-verified this pass) |
| PRODUCT_SALES/STORE floor | $10 |
| Practical min USD | $50/day |
| `budget_mode` | `BUDGET_MODE_DAY` \| `BUDGET_MODE_DYNAMIC_DAILY_BUDGET` \| `BUDGET_MODE_TOTAL` \| `BUDGET_MODE_INFINITE` (forced for `RF_REACH`) |

Source: https://business-api.tiktok.com/portal/docs/campaign-management/budget-verification-ratio-and-value-range-for-each-currency

### Cold-audience-expansion flag

| Field | Value |
|---|---|
| UI name | Smart Targeting (Smart Audience / Smart Interest & Behavior) |
| API fields | `smart_audience_enabled` (boolean, opt-in) + `smart_interest_behavior_enabled` (boolean, opt-in) |
| Default state | Both unset/false — opt-in |
| Deprecated field | `auto_targeting_enabled` — "To be deprecated"; cannot be enabled since June 2024 |

To enable Smart Targeting: pass `smart_audience_enabled=true` AND `smart_interest_behavior_enabled=true`; do NOT pass `auto_targeting_enabled`.

Source: https://business-api.tiktok.com/portal/docs?id=1783164826830849

### Paused-state field

| Field | Value |
|---|---|
| API field | `operation_status` |
| Paused value | `DISABLE` |
| Active value | `ENABLE` (default) |
| Caveat | For `RF_REACH` campaigns, must pass `ENABLE` or omit — cannot pass `DISABLE` |

Source: https://business-api.tiktok.com/portal/docs?id=1739318962329602

### Regulated categories

| Field | Value |
|---|---|
| Crypto allowed? | **Prohibited in the US** under the Financial Services policy — cryptocurrency, exchanges, wallets, NFT trading, crypto advisory all listed as prohibited |
| Some LatAm markets | Argentina, Chile, Colombia, Ecuador, Mexico, Peru, Uruguay — may be allowed with local license + 18+ + TikTok Sales Rep approval |
| Self-serve | Not available for regulated financial — Sales Rep application required |
| For Eco (stablecoin) | Do NOT run in US; other geos require rep + Verified Business Account + 18+ gating + local license + disclaimers |
| `special_industries` enum | `HOUSING`, `EMPLOYMENT`, `CREDIT` (US/CA anti-discrimination — NOT crypto/fintech). Once selected, cannot be changed or re-enabled — only removed. |
| Declaration API field | `special_industries` (US/CA anti-discrimination); crypto/finance is policy-gated at account level, not campaign field |

Source: https://ads.tiktok.com/help/article/tiktok-ads-policy-financial-services

### Audience-size sweet spot
1M–100M (heuristic — TikTok's `/adgroup/audience/estimate/` returns reach estimate but no recommended band).

### Creative specs

| Format | Aspect | Resolution | Duration | Max size | Character limits |
|---|---|---|---|---|---|
| In-Feed Auction (vertical) | 9:16 | ≥540x960 (1080x1920 rec) | ≤600s (9–15s converts best) | 500 MB | Caption white uniform font; no links/@/# |
| In-Feed Auction (square) | 1:1 | ≥640x640 | ≤600s | 500 MB | — |
| In-Feed Auction (horizontal) | 16:9 | ≥960x540 | ≤600s | 500 MB | Heavily letterboxed |
| Spark Ads | inherits from post | inherits | inherits | — | Caption max 4 lines (incl. emojis); requires post authorization code |
| Profile Photo | 1:1 | 98x98 (key element 66x66) | — | 50 KB | — |

### Bid strategy defaults
**Note:** v1.3 deprecated `bid_type`, `deep_bid_type`, `roas_bid`, `optimize_goal` at the campaign level — bidding fields now live on the ad group.

| Context | Ad-group setting |
|---|---|
| Cold, first 2 weeks | Lowest Cost (no bid cap) |
| Mature pixel + known CPA | Cost Cap w/ `conversion_bid_price` ~1.3–1.5× historical CPA |
| App install (SKAN) | oCPM + `billing_event=OCPM`; SAN/MMP integration |
| Reach & Frequency | CPM fixed by inventory contract; not user-controlled |

### Recommendation rules

1. Stablecoin/crypto (Eco) → require Verified Business Account + Sales Rep engagement; DO NOT run US ads; only some LatAm geos permitted with license + 18+ + rep approval.
2. Programmatic create → `operation_status=DISABLE`, verify downstream (ad group + ad + creative + preview), then flip `ENABLE`. Exception: RF campaigns can't be created `DISABLE`.
3. `WEB_CONVERSIONS` or `PRODUCT_SALES` → require Pixel + Events API with ≥50 events/7d on target event before launch.
4. Cold audience, no signal → leave `smart_audience_enabled` and `smart_interest_behavior_enabled` OFF for first 2 weeks; broaden manually (large geo + 18–54 + no interest layering); budget ≥ 20× target CPA/day at ad-group level.
5. Monthly budget <$1,500 (~$50/day floor) → don't run TikTok; redirect.
6. Financial services in US/EU → business verification + per-country legal review before create; disable placements where unlicensed.
7. iOS App Promotion → `campaign_type=IOS14_CAMPAIGN` + `app_promotion_type=APP_INSTALL`; don't mix with non-SKAN ad groups.
8. Premium/reserved inventory → `objective_type=RF_REACH` with `budget_mode=BUDGET_MODE_INFINITE` + fixed flight + rep allowlist.
9. Creative not 9:16 → warn + require vertical as primary (feed is vertical-native).
10. `special_industries` selection → warn: **cannot be changed or re-enabled** post-creation, only removed; US/CA-primary — advertisers registered elsewhere need allowlist to target US/CA with these.

### Unverified / speculative
See CAUTION LIST rows for TikTok.

---

## Cross-platform regulated-categories matrix (finance / crypto)

Verified against official policy pages July 2026. Stablecoins are treated as "cryptocurrency" on every platform — none carry a separate stablecoin-only regime.

| Platform | Declaration / attestation | Crypto & stablecoin allowed? | Geo restrictions | Pre-flight verification | Review SLA delta |
|---|---|---|---|---|---|
| **Meta** | Restricted category — **prior written permission via Business Suite → Authorizations & Verifications**. Separate from `special_ad_categories` API enum (`CREDIT`, `HOUSING`, `EMPLOYMENT`, `ISSUES_ELECTIONS_POLITICS`, `ONLINE_GAMBLING_AND_GAMING`, `FINANCIAL_PRODUCTS_SERVICES` [India-only]). | Yes with permission. Covers exchanges, trading, borrow/lend, wallets (buy/sell/swap/stake), mining software, investment solicitation. Prohibited: ICOs, unlicensed offerings. | License required per country. ~25 accepted regulators: FinCEN MSB or NY BitLicense (US), FCA (UK), BaFin (DE), AMF (FR), FINMA (CH), MAS (SG), FSA (JP), FINTRAC (CA), AUSTRAC/ASIC (AU) + AT/EE/FI/GI/HK/ID/LU/MY/MT/NL/NO/PH/PT/KR/ES/SE/TH/UAE. | Business verification + identity + regulator license upload via Business Suite. | Standard: minutes–24h. Crypto auth: days–multiple weeks (not published). |
| **LinkedIn** | Financial services + cryptocurrency separately Restricted (Nov 18, 2025 policy). No API declaration flag — **advertiser whitelisting via LMS rep**. UK financial-services ads: FCA-authorized only. | Narrow. Exchanges, investment products, wallets allowed **only from authorized advertisers**. Prohibited: specific coins/tokens (incl. individual stablecoin promo), ICOs, DAOs, DeFi, mining, staking, NFTs, guaranteed returns. Consumer-facing crypto: US-only pilot. | Crypto: US, UK, Switzerland, Germany, France, Netherlands, UAE only. Finance-related professionals only. Under-18 excluded. | Prior authorization via LMS rep — no self-serve enrollment. | Standard: <24h. Crypto/financial: manual sales-led — days to weeks. |
| **X** | **Prior certification per category** via ads.twitter.com/help. Certifications don't cross-apply (NFT ≠ crypto ≠ blockchain-game). | Permitted with cert: exchanges, wallets, ATMs/kiosks, crypto debit/credit cards, staking, crypto CFDs, tax calc, DeFi lending/borrowing, DApps, DEXes. No cert required: smart-contract + educational content. Prohibited outright: ICO/IEO/IDExO, mining hardware/software. | Broad ~60-country allowlist. Crypto & DeFi NOT available in: Belgium, Greece, Qatar, Russia, Singapore, Slovenia, Ukraine. Per-country regulator (FinCEN/SEC/CFTC US, FCA UK, BaFin DE, FINMA CH, AMF FR, FSA JP, VASP KR, FSC HK, ASIC/AUSTRAC AU). | Certification submission per category (X's BTC form). No formal biz-verification product. | Not published. Standard hours; category cert several business days to few weeks. |
| **Google Ads** | **"Cryptocurrencies and related products" certification** — apply inside Google Ads (Admin → Policy → Account). Separate from Financial Services / Complex Speculative Products cert. Debt services + CFDs have their own certs. | Permitted with cert: (1) exchanges, (2) software wallets, (3) hardware wallets, (4) coin trusts. No cert needed: educational, tax, mining hardware. Prohibited: ICOs, DeFi trading, crypto loans, IDOs, liquidity pools, unhosted wallets, unregulated dApps, trading signals, aggregators, comparison sites. | ~40 approved locations: US, UK, Canada, Japan, Australia, all EU (MiCA CASP), Switzerland, Bahrain, UAE, Indonesia, Israel, South Korea, Philippines, South Africa, Thailand, Hong Kong. Non-listed = ineligible. | License upload per targeted location. Business/identity verification layered via advertiser-verification program. | Standard 24–48h. 7-day notice before suspension. Cert commonly 3–5 business days. |
| **Reddit** | Restricted — **direct management from a Reddit Sales rep**, not self-serve. Advertiser must be vetted + licensed where legally required. | Yes with sales-rep management. Permitted: exchanges, wallets, crypto cards, staking, lending, mining hw/sw, NFTs, NFT marketplaces, CFDs, forex, brokerage. Prohibited outright: single securities/tokens/coins (so promoting a specific stablecoin is banned), ICOs/IDOs, penny stocks, unlicensed banks/fintechs, payday loans, get-rich-quick, celebrity endorsements, liquidity pools, mixers, bots, DAOs without centralized entity, crypto loans. Ban leverage claims + guarantees. | Case-by-case; no published whitelist. US: FinCEN registration or exemption. | Sales-rep gated; docs at Reddit's discretion. Education/personal-finance-software/industry-events exempt from rep gate. | Standard: same day. Sales-managed crypto onboarding: multi-day to multi-week. |
| **TikTok** | Restricted — **Sales Rep approval**. Self-serve not available at scale. Must comply with local law + local license + disclaimers + 18+ gate. | Approved categories tend to be exchanges/wallets + regulated products. Public help article thin on crypto sub-product allowlist — confirm scope with rep. | Broadly-allowed (directional): AR, CL, CO, EC, MX, PE, UY, CA, US (crypto restricted), BR, CR, SG, AU, FI, FR, DE, IT, NL, IE, UK. Explicitly restricted: DZ, NG, TR, JO, PK. Frequently updated. | Business verification + account whitelisting + financial-services attestation, all via rep. | Not published. Sales-rep onboarding several business days; post-approval ad review comparable to standard. |

### Cross-cutting caveats

- **Stablecoins have no dedicated policy anywhere.** Treated as cryptocurrency via the exchange/wallet listing them. Promoting a specific stablecoin ("Buy USDC") triggers the individual-token prohibition on **Reddit** and **LinkedIn** and ICO-adjacent prohibitions on **Meta** and **Google**. What gets certified is the platform, not the coin.
- **Meta `special_ad_categories: FINANCIAL_PRODUCTS_SERVICES` is India-specific**, not the US/EU crypto gate. Meta's real crypto gate is Authorizations & Verifications with regulator-license upload. Don't conflate in intake.
- **Review SLAs are not published anywhere.** Plan 5–15 business days first-time cert for Meta/Google/X; multi-week for LinkedIn/Reddit/TikTok (all sales-rep-managed).
- **Policy churn:** Meta crypto page updated 2026-04-15; Meta financial-services 2026-04-30; LinkedIn policy revised 2025-11-18. Re-verify before every launch.

### Sources
- Meta Cryptocurrency: https://transparency.meta.com/policies/ad-standards/restricted-goods-services/cryptocurrency-products-and-services/
- Meta Financial: https://transparency.meta.com/policies/ad-standards/restricted-goods-services/financial-services/
- LinkedIn Ads Policy: https://www.linkedin.com/legal/ads-policy
- LinkedIn Crypto Help: https://www.linkedin.com/help/lms/answer/a9401025
- LinkedIn Financial Services Help: https://www.linkedin.com/help/lms/answer/a9357023
- X Financial Services: https://business.x.com/en/help/ads-policies/ads-content-policies/financial-services
- Google Ads Cryptocurrencies: https://support.google.com/adspolicy/answer/14009787
- Google Ads Financial Products: https://support.google.com/adspolicy/answer/2464998
- Reddit Financial/Crypto: https://business.reddithelp.com/s/article/financial-cryptocurrency-products-and-services-policy
- TikTok Financial Services: https://ads.tiktok.com/help/article/tiktok-ads-policy-financial-services