---
name: Create an article and order its publication
description: Draft an article (or import a document), pass the SEO and fit checks, then order publication on chosen sites — the money-spending flow, with its guardrails.
api: openapi/comunicate-top-api-openapi-original.json
operations: [post_articles, post_articles_import, post_media, get_articles_articleId_seo, get_articles_articleId_potrivire, get_balance, post_publications, get_publications_publicationId]
generated: '2026-09-09'
method: generated
---

# Create an article and order its publication

Needs `ARTICLES_WRITE` for drafting, `MEDIA_WRITE` for images, `PUBLICATIONS_WRITE` to order — the only scope that spends credits or money. Over MCP, ordering is additionally refused unless the account owner has enabled the AI spending switch in Integrations.

## Steps

1. **Draft** — `post_articles` (`POST /partner/articles`) with `title`, `contentHtml`, `focusKeyword`, `tags`. Always send an `idempotencyKey` (a locally generated UUID): resending the same key with the same organization returns the already-created article instead of a duplicate. Or import a document with `post_articles_import` (`.docx`, `.doc`, `.odt`, `.rtf`, `.fodt`, `.html`; first file is the document, optional second is the featured image).
2. **Image** — `post_media` (`POST /partner/media`) uploads to the gallery; use the returned URL as `featuredImageUrl`.
3. **Check** — `get_articles_articleId_seo` (`GET /partner/articles/{articleId}/seo`) for the SEO findings (missing alt text is one of them), and `get_articles_articleId_potrivire` for whether it fits the intended campaign type.
4. **Check the balance first** — `get_balance` (`GET /partner/balance`) shows packages with remaining credits and the money balance. Check before ordering; otherwise the integration learns of the shortfall from an error after the article is already built.
5. **Order** — `post_publications` (`POST /partner/publications`) with `articleId`, `siteIds`, `campaignType` (must be among each site's `acceptedCampaigns`), optional `packageTypeId` when several packages cover a site, `extrasBySite`, `scheduledFor`. `licenseNumber` (ONJN) is required when `campaignType` is `CASINO`.
6. **Track** — prefer the `publication.published` webhook (it carries the live URL the moment it appears); polling `get_publications_publicationId` spends your rate limit to learn nothing changed.

## Guardrails

- There is no documented API reversal for a submitted publication — treat `post_publications` as irreversible and confirm intent before calling it.
- Publications are deduped server-side on the article's `version`, which increases only when title or content change — a corrected article re-orders as a new publication; a changed tag does not.
- AI-assisted routes (writing, document import, metadata completion) share a 60/hour organization-wide limit on top of the per-key per-minute limit; back off on `429` using `Retry-After`.
- Errors come as `{message, errors[]}` with real HTTP codes; `400` on insufficient balance means the order was never created.
