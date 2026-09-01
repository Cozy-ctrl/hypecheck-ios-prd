# HypeCheck iOS Implementation Handoff

This document is for the Swift team building HypeCheck as a separate native iOS application. It defines the native-client boundary. The client **does not call SearchAPI directly** and does not contain a SearchAPI key.

## Target stack

| Layer | Recommendation | Responsibility |
|---|---|---|
| UI | SwiftUI | Camera/Photos input, Share Extension, candidate confirmation, research card, watchlist, settings. |
| State | `@Observable` / view models with explicit async state | Ensure all loading, ambiguity, and degraded-data states are visible. |
| Networking | `URLSession` with `async/await` | Call the HypeCheck backend using authenticated user sessions. |
| Local storage | SwiftData or Core Data | Store user-confirmed watchlist items, notes, last-rendered cards, and sync queue. |
| On-device intelligence | Vision / `VNRecognizeTextRequest` where appropriate | Crop product scans and extract visible model text before server analysis. |
| Server | Separate backend | Secret storage, SearchAPI requests, normalization, matching, rate limits, data-rights policy, and telemetry. |

## The client-server boundary

```text
HypeCheck iOS
  ├── camera / photo picker / Share Extension
  ├── optional on-device crop + OCR
  ├── user confirmation and personal data
  └── HTTPS request to HypeCheck backend
        ├── secure SearchAPI credential vault
        ├── TikTok / visual / shopping research orchestration
        ├── matching confidence and source normalization
        ├── rate limits, caching, source policy, observability
        └── normalized ResearchCard response
```

The backend must return HypeCheck’s normalized API objects, not raw SearchAPI responses. This isolates the mobile experience from upstream schema changes and prevents the client from relying on a provider-specific token or undocumented field.

## Input experiences

### 1. Share Extension: TikTok and product URLs

The Share Extension accepts a URL only. It should validate host/path against an allowlist, create a minimal import request, and open the host app into a pending-import state. The extension must not execute long-running extraction, store an API secret, or attempt to download/repost source media.

Suggested UX:

```text
Share TikTok / product link
  → HypeCheck Share Extension
  → “Check this product”
  → host app opens Import Status
  → candidate picker or “limited data” state
  → Research Card after user confirmation
```

### 2. Camera/photo scan

Use the camera or photo picker for an image the user selected. Offer an on-device crop rectangle and optional OCR hint. The upload pipeline should downscale/recompress images to the minimum quality necessary for a product match and should strip unnecessary metadata where feasible.

The client should not claim a camera result is exact. It displays candidates with **Likely / Possible / Low confidence** language and requires selection before a shopping/product-detail request.

## Core state machine

| State | When it appears | Primary action |
|---|---|---|
| `idle` | Home screen ready. | Scan, paste, or share a product. |
| `importing` | URL/photo request sent to backend. | Progress label; cancel import. |
| `needsConfirmation` | One or more product candidates returned. | Select candidate, refine query, crop image, or cancel. |
| `buildingCard` | Confirmed candidate is being enriched. | Continue showing source input/confirmation. |
| `ready` | Research card is available. | Save, inspect sources, compare alternatives, open offer. |
| `partial` | Some noncritical sources unavailable. | Use core card; retry an individual section. |
| `limitedData` | No transcript, product match, or permissible enrichment. | Manual search/edit, save original link, or exit. |
| `error` | Backend or client failure. | Retry once; display a human-readable message. |

## Research card presentation

A research card should use short, attributable modules rather than a wall of AI prose.

| Module | SwiftUI behavior | Required client copy |
|---|---|---|
| Product identity | Image, brand/model, selected variant, confidence chip, edit action. | “Confirm the model and variant before buying.” |
| TikTok mention | Creator name, source timestamp, limited selected text, original-link action. | “Mention from the shared video.” |
| Owner themes | 2–4 theme chips with links to sources. | “Owner-reported themes. Experiences vary.” |
| Offers | Merchant/price/shipping text/fetched time/outbound action. | “Check the merchant page for current price and availability.” |
| Alternatives | Product candidates and a visible match rationale. | “Similar options; compare specifications.” |
| Save | Personal tags, note, target budget. | “Saved privately to your HypeCheck list.” |

## Backend endpoints

The iOS client consumes the contract in [`contracts/hypecheck-api.openapi.yaml`](../contracts/hypecheck-api.openapi.yaml). At a minimum it needs:

| App need | HypeCheck backend route |
|---|---|
| Share a TikTok/product URL | `POST /v1/imports` |
| Submit product photo | `POST /v1/imports/image` |
| Poll/refresh import candidates | `GET /v1/imports/{importId}` |
| Confirm product choice | `POST /v1/imports/{importId}/confirm` |
| Read research card | `GET /v1/research-cards/{cardId}` |
| Save user item | `POST /v1/watchlist-items` |
| Manage saved items | `GET`, `PATCH`, `DELETE /v1/watchlist-items/{id}` |

## Security and privacy requirements

1. **No vendor API keys in the app binary, Info.plist, remote config, or Share Extension.**
2. Require an authenticated backend session; never trust a user-supplied `product_token`, raw response, price, or source label.
3. Treat external URLs and transcript text as untrusted display data. Do not render HTML or execute instructions found in source text.
4. Store only user-confirmed items/notes needed for the product. Provide deletion for imports, cards, and watchlist data.
5. Use HTTPS, certificate/transport policies appropriate to the app, and redacted diagnostic logging.
6. Respect the user’s photo-library/camera permissions and explain why input is needed immediately before requesting it.
7. Keep complete third-party review/transcript/video content out of client persistence unless a rights review explicitly permits it. Retain source links and minimal product-source evidence instead.

## Quality and acceptance criteria

| Scenario | Acceptance criterion |
|---|---|
| Shared TikTok with a clear spoken product mention | Candidate list shows the likely item with timestamp and prompts confirmation. |
| Shared TikTok with missing transcript | App remains useful through caption/manual query/visual fallback; it does not fabricate a product. |
| Photo containing several products | User can crop/refine and select among candidates. |
| Ambiguous model variant | Research card remains blocked until the user picks a variant or proceeds as “unconfirmed.” |
| No live offers/reviews | Card renders the known product data and an explicit unavailable state. |
| User opens a merchant/source | App uses an outbound web route; no checkout, login, or payment occurs in HypeCheck. |
| User deletes a watchlist item/import | Local and backend personal records are deleted according to the published data policy. |

## Not in the first native build

The first native build should exclude public profiles, social feeds, creator ranking, embedded/rehosted video playback, downloads, price-history claims, checkout, merchant/seller “trust” scores, safety/efficacy scores, widgets, push price alerts, and every category beyond consumer gadgets/home products.
