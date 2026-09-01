# HypeCheck Server-Side SearchAPI Request Chains

These are **illustrative backend-only requests**. They use placeholders and omit any production secrets. The iOS app must call the HypeCheck backend, never SearchAPI directly.

## 1. Analyze a user-shared TikTok

The backend receives a TikTok URL from a Share Extension. Query the individual-video metadata first, then request available transcript data for the same URL.

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=tiktok_video' \
  --data-urlencode 'url=https://www.tiktok.com/@creator/video/VIDEO_ID'
```

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=tiktok_transcripts' \
  --data-urlencode 'url=https://www.tiktok.com/@creator/video/VIDEO_ID' \
  --data-urlencode 'language=en'
```

Use video caption, hashtags, mentions, and timestamped transcript segments to propose product entities. Do not treat the absence of a transcript as an error requiring a fabricated answer. Return a limited-data state and allow manual query or photo input.

## 2. Resolve a visual product candidate

HypeCheck sends a minimized, permitted image URL or an internally hosted short-lived image URL from the backend’s secure media pipeline.

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=google_lens' \
  --data-urlencode 'url=https://cdn.example.com/imports/IMAGE_ID.jpg' \
  --data-urlencode 'gl=us' \
  --data-urlencode 'hl=en'
```

If a crop is required, the backend passes the crop information documented for the Lens engine. The client must still show a candidate picker; visual matching is an evidence signal, not a final product decision.

## 3. Search a normalized product query

After transcript/image/URL analysis produces a candidate text query—or after a user selects/refines one—request Google Shopping results. Retain the returned `product_token` with its source/query context only for the follow-on product detail calls that need it.

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=google_shopping' \
  --data-urlencode 'q=Anker Soundcore Motion X600 portable speaker' \
  --data-urlencode 'gl=us' \
  --data-urlencode 'hl=en'
```

The backend returns several normalized `ProductCandidate` records to the mobile client. HypeCheck does not select a product automatically when the query produces more than one plausible model/variant.

## 4. Enrich a user-confirmed product

The current Google Product endpoint requires the `product_token` obtained from Google Shopping. It provides a product detail response with product information, variations, specifications, typical price context, offers, reviews, and more. [Google Product API](https://www.searchapi.io/docs/google-product)

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=google_product' \
  --data-urlencode 'product_token=PRODUCT_TOKEN_FROM_GOOGLE_SHOPPING' \
  --data-urlencode 'gl=us' \
  --data-urlencode 'hl=en'
```

Cache control and any storage/display of third-party content must follow the applicable provider terms. The app card exposes a `generatedAt` time and merchant/source links.

## 5. Expand review context only on demand

Fetch additional review pages only when the user opens the owner-experience module. Request budget and rate limits should prevent a single user from consuming unbounded pagination.

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=google_product_reviews' \
  --data-urlencode 'product_token=PRODUCT_TOKEN_FROM_GOOGLE_SHOPPING' \
  --data-urlencode 'sort_by=most_recent' \
  --data-urlencode 'gl=us' \
  --data-urlencode 'hl=en'
```

The backend should derive a limited number of high-level, attributable owner-theme labels, each linked to sources. It should not copy large volumes of reviews into persistent user-visible storage without rights approval.

## 6. Optional used-market comparison

Use eBay only if the product is confirmed and the user opens the “used / resale” context. In production, favor an authorized marketplace/affiliate/data route for inventory and commerce behavior.

```bash
curl -G 'https://www.searchapi.io/api/v1/search' \
  -H "Authorization: Bearer ${SEARCHAPI_API_KEY}" \
  --data-urlencode 'engine=ebay_search' \
  --data-urlencode 'q=Anker Soundcore Motion X600 portable speaker' \
  --data-urlencode 'ebay_domain=ebay.com'
```

## Backend orchestration pseudocode

```text
analyzeImport(input):
  validate URL/image against server policy
  if TikTok URL:
    metadata = searchapi(tiktok_video, url)
    transcript = attempt searchapi(tiktok_transcripts, url)
    entities = extractCandidateProducts(metadata, transcript)
  if product image:
    entities = searchapi(google_lens, image)
  if product URL:
    entities = parseSafeProductClues(url)

  candidates = searchapi(google_shopping, each normalized entity)
  return import(status=needs_confirmation, candidates, sourceContext)

confirmCandidate(candidate):
  product = searchapi(google_product, candidate.product_token)
  optionalData = loadOnDemand(product_token)
  return normalized ResearchCard(product, optionalData, disclosures, sources)
```

## Operational rules

| Rule | Backend behavior |
|---|---|
| Protect credentials | Read `SEARCHAPI_API_KEY` from a server-side secret manager only. |
| Prevent request storms | Deduplicate imports by normalized URL/hash; throttle per user/device; set upper bounds on pagination. |
| Make source age clear | Return fetched/generation times for offers and research cards. |
| Fail transparently | Return `limited_data` or `partial` instead of inventing results. |
| Respect rights and attribution | Keep source URL/type through normalization and show it in the card. |
| Preserve user control | Delete the user’s saved import/watchlist data on request. |
