---
name: Browse the OLIPOP catalog without enrolling
description: Read OLIPOP's product catalog, flavors, pricing and availability using only the unauthenticated JSON endpoints the store publishes for agents - no token, no UCP profile, no account.
api: https://drinkolipop.com/products.json
surface: json
operations:
  - GET /products.json
  - GET /products/{handle}.json
  - GET /collections/{handle}/products.json
  - GET /search?q={query}&type=product
  - GET /sitemap.xml
generated: '2026-07-31'
method: generated
source: https://drinkolipop.com/llms.txt
---

# Browse the OLIPOP catalog without enrolling

Use this when the user asks what OLIPOP flavors exist, what a can costs, whether a flavor is in stock, or
what is in a variety pack — and you do not need to transact. This is the only OLIPOP commerce surface that
requires no credential of any kind, so reach for it first.

## Rules

- These endpoints are published by the store itself in `/llms.txt` and `/agents.md`. Do not scrape the
  rendered HTML when a `.json` endpoint answers the same question.
- Do **not** use this skill to place an order. Purchase flows live in `olipop-agent-purchase.md` and require
  human approval.
- Respect `/robots.txt`. `/cart/`, `/checkout`, `/checkouts/`, `/orders` and `/account` are disallowed.

## Steps

1. **List the catalog.** `GET https://drinkolipop.com/products.json?limit=250&page=1`. The response is
   `{"products": [...]}`. Paginate with `page` until you get an empty `products` array. Default `limit` is
   30, maximum 250.
2. **Read one product.** If you already know the handle (for example `cream-soda`), skip the list and
   `GET https://drinkolipop.com/products/{handle}.json`. Handles are the URL slugs on
   `https://drinkolipop.com/products/{handle}`.
3. **Scope to a collection.** For a themed subset,
   `GET https://drinkolipop.com/collections/{handle}/products.json`. `collections/all` is everything.
4. **Search.** `GET https://drinkolipop.com/search?q={query}&type=product` for free-text lookups. This
   returns HTML, so prefer step 1 or 2 when you need structured data.
5. **Read variants, not products, for price and stock.** Each product carries a `variants[]` array; price,
   `available`, `sku` and pack size live on the variant, not on the product.

## Gotchas

- Prices in the JSON payload are strings in the shop's currency; there is no currency field on the variant.
  Read `/products/{handle}` HTML metadata or the GraphQL surface if you need explicit currency.
- `body_html` is merchandising copy with markup. Strip it before quoting it to a user.
- There is no rate limit published for these endpoints, but the store asks agents to back off on `429`.
  Apply the same courtesy here.
