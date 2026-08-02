# HYPERLABS

HYPERLABS, Inc. designs and manufactures signal-integrity products for high-speed digital and datacom
test in the United States. Two product classes: ultra-broadband **Components** (amplifiers, baluns, DC
blocks, bias tees, pick-off tees, power dividers, attenuators, terminations, inverters) built for 112 and
224 Gbps PAM-4 applications, and **Instruments** — USB-powered and USB-controlled TDRs, Signal Path
Analyzers, controlled-impedance analyzers and impulse generators with 1-20 channels and rise times as
fast as 35 ps. Founded 1992; engineering and manufacturing in Louisville, Colorado and Beaverton, Oregon.

- Website: https://www.hyperlabs.com/
- GitHub: https://github.com/HYPERLABS
- Support: support@hyperlabs.com · Sales: sales@hyperlabs.com · +1-720-407-6538

## Machine-readable contracts

| Contract | What it is | Where |
|---|---|---|
| **HYPERLABS Web API** | OpenAPI 3.0.1 — 124 paths, 175 operations, 92 schemas. The ASP.NET Core backend behind www.hyperlabs.com: product catalog, faceted filtering, application notes, datasheets, software/DLL downloads, website content, customer accounts with Google/Microsoft sign-in, wishlists and quote requests, plus a role-gated `/v1/admin` surface. Served publicly with Swagger UI. | [`openapi/`](openapi/) · [spec](https://www.hyperlabs.com/api/swagger/v1/swagger.json) · [Swagger UI](https://www.hyperlabs.com/api/swagger/index.html) |
| **radium.v1.Radium** | proto3 gRPC — 22 unary RPCs + 3 server-streaming RPCs for control and waveform acquisition on the TDR11100 Time Domain Reflectometer. MIT licensed, published with generated Python bindings and a worked capture example. Runs on the instrument on TCP 50052 over a plaintext, unauthenticated channel. | [`grpc/`](grpc/) · [repo](https://github.com/HYPERLABS/TDR11100) |

## Artifacts

- [`openapi/`](openapi/) — verbatim copy of the published OpenAPI 3.0.1
- [`grpc/`](grpc/) — verbatim copy of `radium_public.proto`
- [`asyncapi/`](asyncapi/) — AsyncAPI 3.0.0 derived from the three server-streaming RPCs
- [`overlays/`](overlays/) — Overlay 1.0.0 correcting the relative `servers[]` and the misnamed `oauth2` scheme
- [`authentication/`](authentication/) — bearer-JWT profile, federated sign-in, the 121/54 protected/anonymous split, and the unauthenticated gRPC surface
- [`conventions/`](conventions/) — pagination, faceted filtering, identifiers, error envelope, rate-limit signalling
- [`errors/`](errors/) — RFC 7807 problem details plus the gRPC `LocalErrorCode` enum
- [`data-model/`](data-model/) — entity graph derived from the 92 schemas
- [`lifecycle/`](lifecycle/) — versioning, discontinued-products end-of-life path, liveness probe
- [`changelog/`](changelog/) — the published per-product software changelogs, structured
- [`packages/`](packages/) — the ZTDR and XTDR Windows automation DLLs and the Python gRPC bindings
- [`conformance/`](conformance/) — standards posture; RoHS/REACH product-compliance claims
- [`security/`](security/) — TLS/HSTS/DNS probe results
- [`well-known/`](well-known/) — probe record (nothing served; SPA catch-all)
- [`mcp/`](mcp/) — derived candidate tool set; **no** MCP server is published
- [`skills/`](skills/) — three generated agent skills grounded in real operations and RPCs
- [`llms/`](llms/) — generated llms.txt

## Honest gaps

HYPERLABS publishes **no**: `llms.txt`, `security.txt`, any `/.well-known/` document, A2A agent card, MCP
server, GraphQL endpoint, webhooks, status page, SLA, API deprecation policy, public Postman collection,
pricing page (sales are quote-based), or packages on npm / PyPI / NuGet / Maven Central / RubyGems /
Packagist / crates.io / pkg.go.dev.

Notable spec-quality gaps in the published OpenAPI: zero `operationId`s across all 175 operations, zero
operation summaries or descriptions, response schemas on only 10 of 165 successful responses, no 404 /
409 / 422 / 429 / 5xx declared anywhere, and a `securityScheme` keyed `oauth2` that is actually an
apiKey-in-header bearer token.
