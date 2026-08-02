# Olipop

OLIPOP PBC is an Oakland, California prebiotic soda maker founded in 2018 by Ben Goodwin and David Lester.
It runs no traditional developer program, but its Shopify-hosted storefront at
[drinkolipop.com](https://drinkolipop.com/) is a genuinely agent-native commerce surface.

- Agent instructions: https://drinkolipop.com/agents.md (mirrored at `/llms.txt`)
- UCP merchant profile: https://drinkolipop.com/.well-known/ucp
- UCP/MCP commerce endpoint: https://drinkolipop.com/api/ucp/mcp
- Storefront GraphQL: https://drinkolipop.com/api/2025-07/graphql.json

## What is captured here

| Directory | What |
|---|---|
| `llms/` | The store's own `llms.txt` and `agents.md`, verbatim |
| `well-known/` | UCP merchant profile, OIDC discovery, OAuth 2.0 AS metadata, plus the misses |
| `mcp/` | The UCP shopping tool surface and a crosswalk binding it to GraphQL |
| `graphql/` | Full Storefront schema captured by anonymous introspection |
| `authentication/`, `scopes/` | The four coexisting auth models and the published OIDC scopes |
| `conventions/` | Idempotency, pagination, rate limiting, buyer context, human-approval invariant |
| `agentic-access/` | The provider's own published agent access rules, classified per operation |
| `skills/` | Three packaged agent skills, one per surface |
| `conformance/`, `lifecycle/`, `errors/`, `data-model/`, `security/` | Standards posture, versioning, error envelopes, entity graph, domain security |

No OpenAPI, no AsyncAPI, no A2A agent card, no security.txt, no trust center and no first-party SDKs were
found — each of those was probed and the miss is recorded rather than filled in.
