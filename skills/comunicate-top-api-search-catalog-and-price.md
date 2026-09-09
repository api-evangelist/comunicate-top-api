---
name: Search the Comunicate.top catalogue and price a campaign
description: Find publications by niche, authority and price, check what campaign types each accepts, and total the cost — read-only, safe with any key.
api: openapi/comunicate-top-api-openapi-original.json
operations: [get__public_stats, get__public_niches, get__public_niches_slug, get__public_statistici_piata, get_catalog, get_catalog_siteId, get_campaign_types]
generated: '2026-09-09'
method: generated
---

# Search the catalogue and price a campaign

Base URL: `https://app.comunicate.top/api/v1`. The `/public/` routes need no key at all; `/partner/` routes need `Authorization: Bearer bk_live_…` (or an OAuth 2.1 token) with the `CATALOG_READ` scope.

## Without an account (keyless)

1. `get__public_stats` — `GET /public/stats` returns the network size (`publications`, `niches`).
2. `get__public_niches` — `GET /public/niches?locale=ro` lists every niche with its publication count and average authority. Use `locale=en` for English names.
3. `get__public_niches_slug` — `GET /public/niches/{slug}` shows one niche with sample publications: domain, DA, PA, turnaround, price.
4. `get__public_statistici_piata` — `GET /public/statistici-piata` gives the market-wide authority distribution, turnaround times and going price per campaign type.

There is deliberately no public route that returns the whole publication list.

## With a key (full catalogue with prices)

1. `get_campaign_types` — `GET /partner/campaign-types` to learn the campaign types (advertorial, press release, casino, crypto, adult…).
2. `get_catalog` — `GET /partner/catalog` is cursor-paginated (`nextCursor` → resend as `cursor`, `null` on the last page) with the same filters as the UI. Each entry says how it can be paid: `OWN` (your own site), `CREDIT` (covered by a package), `MONEY` (paid from balance), `UNAVAILABLE`.
3. `get_catalog_siteId` — `GET /partner/catalog/{siteId}` for one publication's full detail, including `acceptedCampaigns` and active extras offers (`HOMEPAGE_PLACEMENT`, `FACEBOOK_SHARE`).

## Rules

- Every publication on the network costs the same flat price (45 RON advertorial / 35 RON press release, ex-VAT); casino (100), crypto (150) and adult (100) are priced separately and paid from the money balance.
- Respect `429` + `Retry-After` — the only retryable status. `403` means the key lacks `CATALOG_READ`; do not retry.
