# HYPERLABS

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
