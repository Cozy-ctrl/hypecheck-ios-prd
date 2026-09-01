# HypeCheck — iOS Product Scaffold

HypeCheck is a **source-linked purchase-research companion** for products people discover in short-form video, online stores, or real life. A shopper scans an item, shares a product page, or shares a TikTok link; HypeCheck identifies likely products, surfaces owner-reported trade-offs, shows current offer context, and presents comparable alternatives.

> **Core promise:** Scan it before you buy it.

This repository is intentionally a **product and integration handoff**, not an iOS app. The native Swift application will be developed elsewhere. The material here defines the MVP, server-facing API contract, data-flow requirements, and product/data guardrails.

## Recommended MVP

Launch for **viral consumer gadgets and home products**. The user shares a TikTok or scans a product, confirms the product match, and receives an explainable research card containing product identity confidence, source-linked owner themes, offer context, and alternatives. HypeCheck must not make health, safety, authenticity, fraud, or efficacy judgments.

## Repository guide

| File | Purpose |
|---|---|
| [Product requirements document](docs/PRD.md) | Consumer problem, goals, MVP scope, product flows, and success metrics. |
| [SearchAPI inventory](docs/API_INVENTORY.md) | Relevant endpoints, request chains, data handling, and roadmap placement. |
| [iOS handoff](docs/IOS_HANDOFF.md) | SwiftUI/client responsibilities, UX states, security, and acceptance criteria. |
| [Backend contract](contracts/hypecheck-api.openapi.yaml) | Minimal app-to-backend interface. The backend—not the app—calls SearchAPI. |
| [Data and content guardrails](docs/DATA_AND_CONTENT_GUARDRAILS.md) | Attribution, uncertainty, privacy, and claim boundaries. |
| [Example request chains](examples/searchapi_request_chains.md) | Illustrative backend request sequences using placeholders only. |

## Architecture at a glance

```text
Swift iOS app
  └── HypeCheck backend (authentication, API-key vault, normalization, matching)
        ├── SearchAPI: TikTok Video / TikTok Transcripts
        ├── SearchAPI: Google Lens / Google Shopping / Google Product
        ├── SearchAPI: Google Product Offers / Reviews / Forums
        └── Licensed or authorized product, merchant, affiliate, and marketplace feeds
```

**Never embed a SearchAPI key in the iOS application.** The backend owns API credentials, request budgets, caching, rate control, attribution/rights compliance, and feature fallbacks.

## Status

This is a design scaffold. It contains no production credentials, no copied third-party content, and no app code.

## Key sources

The API inventory cites the relevant SearchAPI documentation. The product decision must additionally comply with current SearchAPI terms and each upstream source’s rights and platform requirements. [SearchAPI Terms](https://www.searchapi.io/legal/terms)
