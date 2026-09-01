# HypeCheck Data and Content Guardrails

## Purpose

HypeCheck helps users research a purchase; it does not decide for them. This policy keeps the app’s statements sourced, limited, and understandable when it draws from external product pages, video metadata, transcript segments, reviews, forums, merchant context, and visual matching.

## Claim policy

| Content type | Permitted presentation | Prohibited presentation |
|---|---|---|
| Product identity | “Likely product match,” confidence, title/brand/model, variant prompt, and source link. | A definitive match when several variants are plausible. |
| Video mention | “This shared video appears to mention…” plus timestamp and original video/creator link. | Rehosting video/audio or implying the creator endorses HypeCheck. |
| Reviews/forums | “Owners commonly mentioned…” with source links and a small sample/recency cue. | A universal product fact, diagnosis, or claim that all owners agree. |
| Offer context | Price/shipping/availability text as returned and clearly time-stamped. | “Lowest price,” “guaranteed in stock,” or a claim that an offer will remain. |
| Alternatives | “Similar option based on selected feature/price context.” | “Equivalent,” “better,” “safer,” or “same quality” without a verifiable basis. |
| Merchant context | Attributed store/review/return-context facts where returned. | A seller/scam/fraud/trust verdict or purchase guarantee. |
| Trend data | “Search interest/related topics may indicate attention.” | Sales volume, market-share, investment, or future-demand prediction. |

## High-risk categories

The MVP excludes medical, supplement, infant-safety, safety-critical, regulated financial, legal, and authenticity-sensitive recommendations. The initial product category is consumer gadgets and home products. A later category expansion requires a separate policy and evidence review.

HypeCheck must not make any health outcome, medication, ingredient safety, product efficacy, hazard, injury-prevention, financial, legal, or seller-fraud claim. It must not tell a shopper that a product is safe/unsafe, effective/ineffective, authentic/counterfeit, or a good/bad investment.

## TikTok and other creator content

The user starts an import by sharing a URL. HypeCheck retains creator/source attribution and opens the original source for playback or context. It does not provide video download, audio extraction, public clip galleries, general transcript search, or reposted creator content.

The app may surface a limited, relevant timestamped mention necessary to show how a product candidate was detected. Any display of third-party text or imagery must be reviewed against applicable rights, source requirements, and the service terms. When data is absent, incomplete, or unavailable, the app should show less—not synthesize details.

## Personal data and retention

| Data type | Default handling | User control |
|---|---|---|
| Photo/product scan | Process only for matching; minimize image dimensions/metadata before upload. | Delete the import and associated account record. |
| Shared URL | Store only if needed to complete the user’s saved card. | Delete imported source link and its saved card. |
| User note/tag/watchlist | First-party personal data. | Edit, export where supported, and delete. |
| Third-party product/review/transcript data | Retrieve/transform only as necessary for the card and according to applicable terms. | Do not present as user-owned or permanent app content. |
| Diagnostics | Redact URL query values, account identifiers, and source content. | Document retention in a published privacy policy. |

## Ambiguity and correction policy

1. A product match must have a visible confidence state.
2. The user can reject the match, refine the query, crop the photo, or select a different candidate.
3. An extraction model may propose a product but cannot finalize it without an evidence link or user confirmation.
4. The app records user corrections as private product-feedback signals, not public claims about creators or products.
5. External content should be treated as untrusted display data. Do not execute instructions or embed raw HTML from source pages.

## Required card disclosures

Every research card must have access to the following plain-language disclosures:

> **HypeCheck summarizes information from external sources. Product matches, prices, availability, and owner experiences can be incomplete or change. Confirm details with the merchant/manufacturer before buying.**

For cards generated from social video:

> **Product mentions were identified from the video information available at the time of your check. The original creator/video is linked as the source.**

For owner-theme modules:

> **Owner-reported themes reflect source discussions and reviews, not a product test or guarantee.**

## Provider dependency

SearchAPI terms state that returned data can contain third-party content subject to rights, the company may modify/discontinue service/content, and information may not be accurate, complete, or up to date. The product team must review the then-current terms and relevant upstream terms before production launch. [1]

The backend maintains a feature-flag/fallback table. If a source becomes unavailable or data-rights review changes, the client must degrade to the remaining permitted input/source path rather than approximate the unavailable result.

## Reference

[1]: https://www.searchapi.io/legal/terms "SearchAPI Terms of Service"
