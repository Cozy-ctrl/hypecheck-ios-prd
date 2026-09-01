# Product Requirements Document: HypeCheck MVP

**Product:** HypeCheck  
**Platform:** iPhone, iOS 17+  
**Product type:** Consumer shopping-research companion  
**Initial category:** Consumer gadgets and home products  
**Primary interaction:** Camera scan or iOS Share Sheet import

## 1. Problem

People increasingly encounter potential purchases in TikTok videos, short-form reviews, product-hauls, creator lists, and unfamiliar online stores. The information required to decide is fragmented: the video may mention a product name only in speech, product variants may be unclear, price/offer context appears elsewhere, and useful owner complaints are scattered across reviews and forums.

The shopper needs a quick answer before buying:

> **What product is this, what trade-offs do owners repeatedly report, what buying options are visible now, and what similar alternatives should I consider?**

HypeCheck creates a source-linked research card from a photo, product URL, or user-shared TikTok URL. It is not a universal recommendation engine, a price-history service, a fraud detector, or a product-safety authority.

## 2. Product promise

**Scan it before you buy it.**

A user can turn an uncertain product discovery into a small, transparent decision workspace in less than a minute. HypeCheck explicitly communicates match confidence, preserves source links, distinguishes sourced owner sentiment from product facts, and lets the shopper save a product or alternative to a personal watchlist.

## 3. Goals and non-goals

| Goals | Non-goals |
|---|---|
| Identify likely consumer gadgets/home products from a user image, product link, or TikTok link. | Search the entire TikTok corpus by transcript or quote. |
| Extract product names, models, brands, and mentions from an individual user-shared TikTok when available. | Download, rehost, publish, or build a public index of creator video/music/transcript content. |
| Present product confidence, current offer context, reviewed trade-off themes, and alternatives with source links. | Guarantee the lowest price, an exact match, current inventory, product authenticity, or seller trustworthiness. |
| Create private saved-product/watchlist value with user notes and alerts based on permitted data. | Make medical, safety, efficacy, legal, financial, or fraud decisions for the shopper. |
| Demonstrate a repeat scan/save/retrieve loop. | Build a social network, public reviews community, checkout system, or generic AI shopping chatbot. |

## 4. Personas

| Persona | Situation | Job to be done |
|---|---|---|
| Viral-gadget shopper | Sees a compelling product review or haul on TikTok. | Identify the actual model and decide whether to investigate or save it. |
| Gift researcher | Sees a product online but needs alternatives within a budget. | Turn a discovery into a short, source-linked comparison. |
| Home-product buyer | Sees a device in-store or in a video and does not know its exact model/variant. | Scan it, confirm the model, and understand trade-offs before purchasing. |

## 5. MVP input modes

### A. TikTok Share Sheet import

The user selects **Share → HypeCheck** from a TikTok URL. The app sends the URL to HypeCheck’s backend. The backend requests allowed metadata/transcript enrichment, extracts candidate product entities from caption, hashtags, mentions, and timestamped transcript text, then returns candidates to the user for confirmation.

This input is especially valuable for product-review, product-haul, “TikTok made me buy it,” gift-guide, and comparison videos. The UI must show the creator name, video title/caption where provided, and a link back to the original video. The application stores the minimum necessary metadata and never republishes the video or music.

### B. Camera/photo scan

The user takes or selects an image. The client performs basic on-device crop and text recognition where appropriate, then securely uploads a resized scan to the backend. The backend uses visual-product discovery and returns a ranked candidate list. The user must select or confirm the product before HypeCheck presents detail data.

### C. Product-page Share Sheet import

The user shares a retail product URL. The backend parses the URL for product title/model clues and queries product-discovery services. The same confirmation step occurs before results are shown.

## 6. Product research card

A confirmed product opens a compact research card. Empty or uncertain data must be represented plainly—not filled with unsupported AI guesses.

| Section | MVP behavior | Required disclosure |
|---|---|---|
| Product match | Image, title, brand/model, selected variant, and confidence. | “Confirm the model/variant before purchase.” |
| Mentioned in video | Timestamped creator quote/mention snippets selected by the user or detected with confidence. | Original creator/video link; no rehosted playback. |
| Owner themes | 2–4 recurring, plainly worded sentiment themes linked to source results. | “Owner-reported themes; experiences vary.” |
| Offer context | Visible merchant offers, shipping/availability detail where returned, and click-through links. | “Offers can change; check merchant checkout.” |
| Alternatives | 3 comparable candidates based on stated model/spec/price criteria. | “Similarity is approximate; compare specifications.” |
| Save | Save user-confirmed product, notes, target budget, and alternatives. | User controls deletion/export of personal data. |

## 7. TikTok product-mention flow

```text
User shares TikTok URL
  → backend requests TikTok video metadata
  → backend requests transcript where available
  → entity extractor proposes brands/models/categories + timestamps
  → visual candidate search uses thumbnail/frame only when permitted and needed
  → user confirms a candidate or searches manually
  → product search yields a reusable product token
  → product detail, reviews, offers, and forum context enrich the card
  → user saves the product, alternative, note, or source link
```

### Candidate-extraction rules

1. The system distinguishes **explicit spoken/captioned product names** from visual inferences.
2. It shows visual matches as **possible matches**, never as certain identifications without user confirmation.
3. If no single product is confidently found, the app returns a short list or asks the user to crop/describe the item.
4. It does not infer products from private data, face analysis, or personal characteristics.
5. It does not claim a creator was paid, affiliated, deceptive, or correct unless an official source directly establishes a disclosure.

## 8. Key user stories

| ID | User story | Acceptance criterion |
|---|---|---|
| US-01 | As a shopper, I can share a TikTok product review into HypeCheck. | A pending analysis appears with video/creator attribution and a clear status. |
| US-02 | As a shopper, I can confirm the product that HypeCheck believes was mentioned. | No research card is finalized until I select a candidate or search manually. |
| US-03 | As a shopper, I can see owner-reported trade-off themes with sources. | Each theme has sources or is omitted; no unsupported claim appears. |
| US-04 | As a shopper, I can open currently visible purchase options. | Each offer uses a clearly labeled outbound link and records no checkout credentials. |
| US-05 | As a shopper, I can save an item with a note and later retrieve it. | Saved content retains the original link and user data can be deleted. |
| US-06 | As a shopper, I can scan a physical product or packaging. | The app presents multiple possibilities when the visual match is ambiguous. |

## 9. Success metrics

The first release is a learning product. It should not optimize for superficial downloads before validating repeat value.

| Metric | 30-day target | Interpretation |
|---|---:|---|
| Share/scan → product-confirmation rate | ≥ 55% | Identification flow creates enough confidence to proceed. |
| Confirmed research card → save/alternative/merchant-link rate | ≥ 30% | The card informs a real shopping action. |
| Week-4 repeat checkers among activated users | ≥ 20% | HypeCheck is more than a novelty scanner. |
| Manual correction rate | ≤ 10% of confirmed-product journeys | Entity/visual matching is sufficiently reliable for the category. |
| Unsupported-claim report rate | < 1% of cards | Guardrails and source linking work. |

## 10. Explicit risks

The initial data layer is dependent on third-party content and search platforms. SearchAPI may change or discontinue service/content and its data may be subject to third-party rights; the backend needs request budgets, fallbacks, attribution review, and a provider-agnostic normalization layer. [SearchAPI Terms](https://www.searchapi.io/legal/terms)

The user may share content that has missing/poor captions, multiple items, affiliate links, or sponsored content. The product must gracefully return a manual-search or “limited information” state. Inaccurate identification is a worse outcome than incomplete identification.

## 11. MVP release decision

Proceed beyond a TestFlight launch only if the core gadget category reaches the 30-day metrics, users return to saved products, and the product/data team confirms a durable/authorized route for the core price/inventory and outbound-commerce experience. If users scan once but rarely save/retrieve, pivot to a more focused job such as product-replacement search or purchase-decision cards for a single high-interest category.
