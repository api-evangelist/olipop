---
name: Buy OLIPOP as an agent over UCP/MCP
description: Run the store's published agent commerce flow - discover, search, cart, checkout, fulfill, complete - against the Universal Commerce Protocol MCP endpoint OLIPOP serves, with the mandatory human buyer-approval step before payment.
api: https://drinkolipop.com/api/ucp/mcp
surface: ucp-mcp
operations:
  - search_catalog
  - get_product
  - create_cart
  - update_cart
  - create_checkout
  - update_checkout
  - complete_checkout
  - get_order
generated: '2026-07-31'
method: generated
source: https://drinkolipop.com/llms.txt
---

# Buy OLIPOP as an agent over UCP/MCP

OLIPOP's storefront implements the Universal Commerce Protocol (UCP) shopping service over MCP JSON-RPC.
This skill is the store's own published agent flow. Follow it exactly — especially step 6.

## Hard rules (published by the store, not by us)

- **Checkout requires human approval.** You must not call `complete_checkout` without explicit,
  contemporaneous buyer consent at the moment of payment. If you cannot obtain it, stop and route the
  purchase through the Shop skill (`https://shop.app/SKILL.md`) and Shop Pay instead.
- **No scripted checkout.** `/robots.txt` prohibits form fills, browser automation and end-to-end flows
  that finalize payment.
- **Identify yourself.** Every call must carry `meta["ucp-agent"].profile` — a resolvable URL to your
  platform's own UCP profile document, sent as the `UCP-Agent` HTTP header. Anonymous calls fail closed
  with JSON-RPC `-32001` / `invalid_profile_url` before any tool is exposed.
- **Back off on 429.** The endpoint is rate-limited per IP.

## Steps

1. **Discover.** `GET https://drinkolipop.com/.well-known/ucp`. Confirm the store's `ucp.version` is one you
   support (currently `2026-04-08`; `2026-01-23` is also accepted) and read `capabilities` and
   `payment_handlers`. Do not hardcode the version — negotiate it from this document every session.
2. **Search.** Call `search_catalog` with `meta` and a `catalog` search request. Always pass
   `context.address_country` and `context.currency` in the buyer context, or prices and availability will be
   wrong. Use `get_product` or `lookup_catalog` to resolve a specific item or variant.
3. **Cart.** Call `create_cart` with the selected variants and quantities. Use `update_cart` to change
   lines, attributes, notes or discount codes. Re-read with `get_cart` after every mutation — do not assume
   your local view is authoritative.
4. **Checkout.** Call `create_checkout` from the cart. Keep the returned checkout `id`.
5. **Fulfill.** Call `update_checkout` with the shipping address and delivery method. This store declares
   `allows_multi_destination.shipping: false`, so one checkout ships to exactly one address.
6. **Get consent, then complete.** Present the final total, shipping method and address to the human and
   obtain explicit approval *now*, not earlier in the conversation. Only then call `complete_checkout`.
   This call **requires** `meta["idempotency-key"]` (a UUID) — generate one per logical purchase attempt and
   reuse the same value on every retry, never a fresh one.
7. **Confirm.** Call `get_order` with the resulting order id to read back status and fulfillment.

## Idempotency

`meta["idempotency-key"]` maps to the HTTP `Idempotency-Key` header and is **required** on
`complete_checkout`, `cancel_checkout` and `cancel_cart`. It is optional but recommended on every other
write. Generating a new key on retry will double-charge the buyer; that is the single most damaging mistake
an agent can make on this surface.

## Errors

Errors come back as JSON-RPC 2.0 error objects: `error.code`, `error.message`, and an `error.data` object
carrying a machine-readable `code`, human-readable `content`, and often a `continue_url`. When you get a
`continue_url`, hand it to the human rather than retrying blindly — it is the store's escape hatch back to a
browser flow. See `errors/olipop-problem-types.yml`.
