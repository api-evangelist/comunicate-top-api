---
name: Order a written article and track it to delivery
description: Commission an article from the platform's writing service, follow its states, cancel while still possible, and pick up the delivered article.
api: openapi/comunicate-top-api-openapi-original.json
operations: [get_balance, post_redactare, get_redactare, get_redactare_orderId, post_redactare_orderId_anuleaza, get_articles_articleId]
generated: '2026-09-09'
method: generated
---

# Order a written article and track it to delivery

Needs `ARTICLES_READ`/`ARTICLES_WRITE`. Writing orders are paid from the **money balance, not credits**; the price is reserved when the order enters and consumed at delivery. Over MCP this is a spending tool, behind the account's AI spending switch.

## Steps

1. **Balance first** — `get_balance`: without balance the order is rejected with `400` and never created.
2. **Order** — `post_redactare` (`POST /partner/redactare`) with the topic (`tema`) and, critically, `fapte`: the only field that measurably changes quality. Only what appears in `fapte` may appear in the article as a number or a name. Send an `idempotencyKey`.
3. **Track** — the response is the order with `status: "NOUA"`. States: `NOUA` → `IN_LUCRU` → `LIVRATA` (with `articleId` and `chargedCents`), or `REFUZATA` / `ANULATA` / `ESUATA` — in those three the reserved amount returns to the balance in full. Prefer the `redaction.delivered` webhook (carries `orderId` + `articleId`); a human review happens between request and delivery, so the duration cannot be predicted. Poll `get_redactare_orderId` sparingly, list with `get_redactare` (cursor pagination).
4. **Cancel (window)** — `post_redactare_orderId_anuleaza` works only while the order is `NOUA`; once picked up you get `409` and closing it is the platform's call.
5. **Pick up** — on `LIVRATA`, read the article via `get_articles_articleId`. `articleId` is set only at delivery.
