# HypeCheck SearchAPI Inventory

## Integration principle

HypeCheck should use SearchAPI to **identify and enrich** a shopper’s research journey. It must not expose its API key in the iOS client, copy third-party content into a public corpus, or depend on a single scraped response for an irreversible claim. The backend owns authentication, request quotas, normalization, matching, source attribution, caching policy, and graceful failure behavior.

The request chain begins with one of three user-authorized inputs: a TikTok URL, product-page URL, or product image. It ends only after the user confirms a product candidate. That confirmation is necessary because product variants, incomplete transcripts, and image ambiguity are normal—not exceptional—cases.

## MVP endpoint set

| Endpoint | Engine | Classification | HypeCheck function | Required input / dependency |
|---|---|---|---|---|
| TikTok Video | `tiktok_video` | **MVP essential** for TikTok import | Obtain caption, hashtags, mentions, author/video metadata, and available subtitle context from an individual shared video. | Full TikTok URL, or `video_id` + `username`. [1] |
| TikTok Transcripts | `tiktok_transcripts` | **MVP essential** for TikTok import | Obtain timestamped spoken-caption segments and available language tracks so a product mention can be proposed with a source moment. | Full TikTok URL, or `video_id` + `username`; selected language optional. [2] |
| Google Lens | `google_lens` | **MVP essential** for photo/visual fallback | Identify possible products from a user image or an appropriately handled video thumbnail/frame; supports crop coordinates. | Publicly retrievable image URL and optional crop/localization. [3] |
| Google Shopping | `google_shopping` | **MVP essential** | Turn a user-confirmed title/model into candidate products, visible price/merchant context, and a product token for detail calls. | Search query, geography/language, and selected `product_token` from returned shopping result. [4] |
| Google Product | `google_product` | **MVP essential** | Retrieve a confirmed product’s title, brand, variations, specifications, typical price context, offers, rating/review summary, and related context. | A `product_token` minted by Google Shopping. The legacy `product_id`/`prds` input was removed for this endpoint in May 2026. [5] |
| Google Product Reviews | `google_product_reviews` | **MVP optional** | Retrieve more review text when card-level owner themes require more evidence. | `product_token`; optional rating, relevance/recency sort, and pagination token. [6] |
| Google Product Offers | `google_product_offers` | **MVP optional** | Expand offer detail after the user requests “See offers.” | Product token from the preceding shopping/product flow. [7] |
| eBay Search / Product | `ebay_search` / `ebay_product` | **MVP optional** | Add used/secondary-market alternatives for a clearly identified product. | Product title/model or item/ePID from an upstream result. [8] [9] |
| Google Forums | `google_forums` | **MVP optional** | Supply attributed discussion links/snippets for a “what owners discuss” section. | Product model/brand query, preferably user-confirmed. [10] |

## Phase 2 enrichment

| Endpoint | Engine | Use only when | Product boundary |
|---|---|---|---|
| Google About This Store | `google_about_this_store` | A user opens a merchant offer and wants seller/service context. | Show attributed merchant context, review themes, and return/shipping data where returned; never label a merchant “safe,” “unsafe,” or fraudulent. [11] |
| Google Trends | `google_trends` | A product’s cultural/trend context meaningfully helps explain why it is viral. | Use as optional context, with attribution; never claim market size, product quality, or future price movement. [12] |
| YouTube Video | `youtube` | A product detail result surfaces a relevant, attributable long-form review. | Deep-link to source and show only needed title/channel/key-moment context. [13] |
| YouTube Transcripts | `youtube_transcripts` | The user asks for timestamped long-form review rewatch points. | Keep snippets limited and source-linked; do not recreate or redistribute reviews. [14] |
| Google Images | `google_images` | Visual candidate disambiguation needs more references. | Treat every result as a possible visual match; preserve source links. [15] |
| Google About This Domain | `google_about_this_domain` | A later version tests generic web-store context. | Keep out of MVP because domain context is not sufficient for seller/trust conclusions. [16] |

## Explicit non-endpoints / exclusions

There is no separate `google_product_specs` engine required for HypeCheck. The **Google Product** response itself includes a `specifications` field, along with product detail, variations, offers, reviews, and related materials. [5]

HypeCheck should exclude any general “is this a scam?” score, authenticity verdict, seller-fraud label, health/safety conclusion, medical efficacy claim, price-history graph without a permitted historical data provider, or any claim derived solely from user-generated review text.

## Core request chains

### 1. TikTok product discovery

```text
User shares TikTok URL
  → `tiktok_video` (caption, hashtags, mentions, creator/video metadata)
  → `tiktok_transcripts` (timestamped text; only when available)
  → backend entity extractor proposes brand/model/category + source timestamps
  → if uncertain: `google_lens` with a permissible image input or prompt user to edit/query
  → user confirms candidate
  → `google_shopping` (candidate search)
  → select `product_token`
  → `google_product` (detail card)
  → optional: `google_product_reviews`, `google_product_offers`, `google_forums`, `ebay_search`
```

**Important:** The TikTok transcript endpoint retrieves a transcript for the specified video only. It is not a global TikTok spoken-word search index. [2]

### 2. Camera/photo product discovery

```text
User captures/selects photo
  → on-device crop/OCR first where useful
  → upload minimized image to backend
  → `google_lens` (visual candidates/extracted text)
  → user confirms or refines the product
  → `google_shopping`
  → `google_product`
  → optional review, offer, forum, or eBay enrichment
```

### 3. Product URL research

```text
User shares merchant/product URL
  → backend parses safe title/model hints (do not crawl arbitrary user credentials/content)
  → `google_shopping` with normalized query
  → user confirms candidate
  → `google_product`
  → optional `google_product_offers` / `google_about_this_store`
```

## Normalized backend model

The mobile client should never consume vendor-specific result shapes. The backend translates each source into an app-facing research model and labels provenance.

| Object | Fields | Source-of-truth rule |
|---|---|---|
| `ProductCandidate` | ID, title, brand, variant, image URL, match confidence, source IDs | User confirmation is required before use as the selected product. |
| `VideoMention` | video ID/link, creator, timestamp range, extracted entity, extraction confidence | Retain original link and distinguish spoken/captioned mention from image inference. |
| `ResearchCard` | product summary, selected variant, key specifications, source links, generated-at time | Do not imply complete/current truth. |
| `OwnerTheme` | theme label, source count, source excerpts/links, recency band | Present as attributed owner-reported context, not a product fact. |
| `Offer` | merchant, price text/value, shipping detail, availability text, outbound URL, fetched-at time | Offer is informational only; merchant checkout remains authoritative. |
| `Alternative` | title, match rationale, price context, feature context, source link | Similarity/fit are approximate and must be explained. |
| `WatchlistItem` | user selection, personal note, target price, created/updated time | This is first-party user data. |

## Reliability and fallback matrix

| Failure / ambiguity | Backend action | Mobile UX |
|---|---|---|
| TikTok transcript absent | Continue with caption/hashtags/mentions and ask user for a product name or image crop. | “This video has limited caption data. Search manually or scan the item.” |
| Multiple product entities in a video | Return separate timestamped candidates. | “Which item do you want to check?” |
| Visual match uncertainty | Provide 2–5 candidates or a crop/retry flow. | Show confidence and require selection. |
| No Google Shopping product token | Allow manual query refinement; do not call product-detail endpoint. | Search refinement and “not found” state. |
| Product token expires / request fails | Re-run candidate search once from normalized user query; do not loop indefinitely. | “Refresh product match” action. |
| Reviews/offer data absent | Render the core product card without that section. | “No current review/offer context found.” |
| External source outage / rate budget exceeded | Serve a recent, clearly timestamped permitted cache only if policy allows; otherwise show reduced card. | “Some live sources are unavailable. Try again later.” |

## Server-only credential policy

The backend submits the SearchAPI key in an Authorization header, never to the iOS client. API quotas, per-user throttles, error telemetry, and any data-retention controls are enforced server-side. SearchAPI documents bearer-token authentication for the relevant engines and an enterprise zero-retention option for supported requests. [1] [2]

## References

[1]: https://www.searchapi.io/docs/tiktok-video-api "SearchAPI TikTok Video API"
[2]: https://www.searchapi.io/docs/tiktok-transcripts-api "SearchAPI TikTok Transcripts API"
[3]: https://www.searchapi.io/docs/google-lens-api "SearchAPI Google Lens API"
[4]: https://www.searchapi.io/docs/google-shopping-api "SearchAPI Google Shopping API"
[5]: https://www.searchapi.io/docs/google-product "SearchAPI Google Product API"
[6]: https://www.searchapi.io/docs/google-product-reviews "SearchAPI Google Product Reviews API"
[7]: https://www.searchapi.io/docs/google-product-offers-api "SearchAPI Google Product Offers API"
[8]: https://www.searchapi.io/docs/ebay-search-api "SearchAPI eBay Search API"
[9]: https://www.searchapi.io/docs/ebay-product-api "SearchAPI eBay Product API"
[10]: https://www.searchapi.io/docs/google-forums-api "SearchAPI Google Forums API"
[11]: https://www.searchapi.io/docs/google-about-this-store-api "SearchAPI Google About This Store API"
[12]: https://www.searchapi.io/docs/google-trends-api "SearchAPI Google Trends API"
[13]: https://www.searchapi.io/docs/youtube-video "SearchAPI YouTube Video API"
[14]: https://www.searchapi.io/docs/youtube-transcripts "SearchAPI YouTube Transcripts API"
[15]: https://www.searchapi.io/docs/google-images-api "SearchAPI Google Images API"
[16]: https://www.searchapi.io/docs/google-about-this-domain-api "SearchAPI Google About This Domain API"
