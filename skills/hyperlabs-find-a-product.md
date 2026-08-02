---
name: Find a HYPERLABS product and its specifications
description: >-
  Navigate the HYPERLABS catalog over its public web API — pick a product class, narrow by category and
  facets, resolve a specific SKU, and pull the datasheets, application notes and software that belong to
  it. Read-only and anonymous; no token required.
api: openapi/hyperlabs-web-openapi-original.json
base_url: https://www.hyperlabs.com/api
auth: none
operations:
  - GET /v1/product-classes
  - GET /v1/categories
  - GET /v1/products/filter/parameters
  - POST /v1/products/filter
  - GET /v1/products
  - GET /v1/products/{topSku}
  - GET /v1/application-notes
  - GET /v1/software
---

# Find a HYPERLABS product and its specifications

HYPERLABS publishes an OpenAPI 3.0.1 definition at
`https://www.hyperlabs.com/api/swagger/v1/swagger.json` with a Swagger UI at
`https://www.hyperlabs.com/api/swagger/index.html`. The catalog read operations below are anonymous —
they declare no security requirement and return JSON without a token.

**Before you start, read this.** The spec declares no `operationId` on any of its 175 operations, so
operations are named here by METHOD + path, which is unambiguous. It also declares a response schema on
only 10 of 165 successful responses, so you must inspect the live JSON rather than trusting the spec for
response shape. Identifiers are RFC 4122 GUIDs.

## Steps

1. **Pick a product class.** `GET /v1/product-classes` returns the two top-level classes with their id,
   `name` (`Components`, `Instruments`), `title`, `subtitle`, `description`, `hasCategories` and
   `publishingStatusCode`. Keep the `id` — everything downstream keys off it.

2. **Narrow to a category.** `GET /v1/categories` accepts `ProductClassIds`, `ParentCategoryId` and
   `IsTopLevelOnly`. Categories are a self-referential tree (`parentCategoryId`), so walk down rather
   than assuming one level.

3. **Ask what is filterable before you filter.** `GET /v1/products/filter/parameters` returns the
   facets available for the class/category selection. Do not guess parameter names — the facet set is
   data, not a fixed schema, and each selected facet is typed by the `FilterParameterCode` enum.

4. **Run the faceted search.** `POST /v1/products/filter` with a `FilterDto`:
   `{search, productClassIds[], categoryIds[], parameters[]}`, where each entry of `parameters[]` is a
   `FilterSelectedParameterDto` `{code, valueIds[]}`. For a simpler query, `GET /v1/products` accepts
   `ProductClassIds`, `CategoryIds`, `TagIds`, `SoftwareIds`, `IsNew`, `IsFeatured` and `IsPromoted` as
   repeated query parameters.

5. **Resolve a known part number.** `GET /v1/products/{topSku}` resolves by SKU (for example `HL1101`).
   Note that `GET /v1/products/{id}` and `GET /v1/products/{topSku}` are the same path template in the
   spec — pass a GUID for the first and a SKU for the second.

6. **Pull the supporting material.** `GET /v1/application-notes` lists application notes with the
   products each applies to; `GET /v1/software` lists instrument software with its `resources[]`
   (installer, user manual, changelog) and `dlls[]` (automation DLL, DLL manual, C++ sample source) and
   the `products[]` it supports.

## Rules

- **Paging is not universal.** The public catalog endpoints return whole unpaged arrays. Only the four
  admin collection endpoints take `PageNo` / `PageSize` / `SortBy` / `SortDesc` / `Search`. Do not send
  paging parameters to `/v1/products` and expect them to apply.
- **Nothing here is idempotency-keyed** because nothing here writes. Never escalate from this skill into
  `POST /v1/request-quote`, `POST /v1/contact-messages`, `POST /v1/wishlist/quote` or
  `POST /v1/wishlist/share` on the user's behalf — those send real email to HYPERLABS sales and require a
  `recaptchaToken` the user must produce.
- **Errors.** 401/403 mean you have wandered onto a protected operation — the catalog reads are
  anonymous. Errors that do carry a body use RFC 7807 `ProblemDetails`
  (`type`/`title`/`status`/`detail`/`instance`), served as `application/json`. See
  `errors/hyperlabs-problem-types.yml`; 404, 409, 422, 429 and 5xx are undeclared in the spec, so handle
  them defensively.
- **Rate limits are undocumented.** No 429 is declared, but the hyperlabs.com front-end handles 429 and
  reads `RateLimit-Reset`, `X-Rate-Limit-Reset` and `X-RateLimit-Reset`. Back off on 429 and honour
  whichever of those headers appears.
- **Check for discontinuation.** `GET /v1/products/discontinued` lists end-of-life products; the product
  model carries `replacementProductId`, so always report the successor rather than a dead SKU.
- The API is not marketed as a developer product and publishes no terms of use for third-party
  consumption. Treat it as read-only public data and keep request volume modest.
