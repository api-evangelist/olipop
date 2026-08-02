---
name: Query the OLIPOP storefront with GraphQL
description: Use the Shopify Storefront GraphQL API served on drinkolipop.com for structured catalog, collection, content and cart queries that the flat JSON endpoints cannot answer.
api: https://drinkolipop.com/api/2025-07/graphql.json
surface: graphql
operations:
  - product
  - productByHandle
  - products
  - collection
  - collections
  - search
  - predictiveSearch
  - productRecommendations
  - blog
  - articles
  - page
  - menu
  - shop
  - localization
  - publicApiVersions
  - cartCreate
  - cartLinesAdd
  - cartLinesUpdate
generated: '2026-07-31'
method: generated
source: graphql/olipop-storefront.graphql
---

# Query the OLIPOP storefront with GraphQL

Reach for this when the flat JSON endpoints are too coarse: you need selected fields, cursor pagination,
localized pricing, collection filters, blog/page content, metafields, or a cart you intend to hand back to
a human in a browser.

The full schema captured from this endpoint is in `graphql/olipop-storefront.graphql` — 422 types, 35 root
queries, 41 mutations. Grep it before composing a query; do not guess field names.

## Auth

- Schema **introspection** answered anonymously when this artifact was captured.
- **Data reads and all mutations require** an `X-Shopify-Storefront-Access-Token` header. OLIPOP does not
  publish a public storefront token, so you need one issued for your integration.
- Customer-scoped queries additionally need a `customerAccessToken` argument, minted by
  `customerAccessTokenCreate`. Customer accounts are OIDC-backed — see `authentication/olipop-authentication.yml`.

## Version

The path segment is the API version. `publicApiVersions` tells you what is currently served:

```graphql
{ publicApiVersions { handle displayName supported } }
```

This artifact was captured against `2025-07`; `2026-07` was the latest supported version at capture time
and `2026-10` was a release candidate. Pin a supported dated version — never `unstable`.

## Steps

1. **Find products.** `products(first: 50, query: "...")` or `productByHandle(handle: "cream-soda")`.
   Price, availability and pack size live on `variants`, not on `Product`.
2. **Paginate properly.** Connections follow the GraphQL Cursor Connections spec: request
   `pageInfo { hasNextPage endCursor }` and pass `after: $endCursor`. Never loop on `first:` alone.
3. **Localize.** Wrap queries in the `@inContext(country: US, language: EN)` directive for correct currency
   and availability instead of passing context arguments.
4. **Browse collections.** `collection(handle:)` then `products` on it; `productRecommendations` for
   related items.
5. **Read content.** `blog`, `articles`, `page`, `menu`, `metaobjects` cover the editorial surface that has
   no UCP tool at all.
6. **Build a cart.** `cartCreate`, then `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove`. The returned
   `checkoutUrl` is a human-completable browser URL — hand it to the user rather than automating it.

## Errors

Two layers: a transport-level `errors[]` array with `extensions.code`, and typed `userErrors[]` on every
mutation payload (all implementing the `DisplayableError` interface). Always select `userErrors { field
message code }` on mutations — a mutation can return HTTP 200 with a populated `userErrors` and a null
payload.

## Cost

Responses carry `extensions.cost.requestedQueryCost`. Shopify applies a calculated-cost leaky bucket; keep
selection sets tight and expect throttling on deep nested connections.
