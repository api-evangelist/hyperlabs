---
name: Resolve the right HYPERLABS software and automation package for an instrument
description: >-
  Given a HYPERLABS instrument or SKU, find the software that drives it, the automation DLL or gRPC
  interface that scripts it, the manual and sample source that document it, and the changelog that says
  what changed — all from the public web API.
api: openapi/hyperlabs-web-openapi-original.json
base_url: https://www.hyperlabs.com/api
auth: none
operations:
  - GET /v1/software
  - GET /v1/software/{name}
  - GET /v1/products/{topSku}
  - GET /v1/products
  - GET /v1/products/discontinued
---

# Resolve the right HYPERLABS software and automation package for an instrument

Every HYPERLABS instrument is driven by a named Windows application, and most of them also ship a
programmatic interface. The catalog endpoint that lists them is anonymous and returns the download,
manual, changelog and sample-source URLs directly.

## Steps

1. **List what exists.** `GET /v1/software` returns every software package with `id`, `name`, `title`,
   `description`, `keyFeatures`, `notes`, `images`, `resources[]`, `dlls[]` and `products[]`. As of this
   capture the set is XTDR, Zcoupon, ZTDR and TDR11100.

2. **Resolve by name if you already know it.** `GET /v1/software/{name}` accepts the URL-encoded display
   name including the trademark symbol — `ZTDR%E2%84%A2`, `XTDR%E2%84%A2`, `Zcoupon%E2%84%A2` — or the
   plain name where there is no symbol (`TDR11100`). The bare ASCII form of a trademarked name returns
   404; encode the `™`.

3. **Go the other direction from a SKU.** `GET /v1/products/{topSku}` resolves a part number
   (for example `HL1101`), and `GET /v1/products?SoftwareIds=…` lists every product a given software
   package supports. `software[].products[]` carries the same link from the software side.

4. **Separate `resources[]` from `dlls[]`.** `resources[]` holds the end-user assets — the installer
   zip, the user manual PDF, and the changelog text file. `dlls[]` holds the automation assets — the DLL
   zip, the DLL manual PDF, and a C++ test program with source. If the caller wants to script the
   instrument, `dlls[]` is the answer; if they want to run it by hand, `resources[]` is.

5. **Pick the right automation path per instrument.**
   - **HL1101 Ruggedized USB TDR** → the **ZTDR DLL** (currently 2.1.0). The DLL automates acquisition
     and logging from any test environment; a DLL guide and C++ test program with source are published
     alongside it.
   - **HL22xx / HL52xx Signal Path Analyzers** → the **XTDR DLL** (currently 2.6.0), which adds
     programmatic insertion-loss (S21) and return-loss (S11) measurement on top of TDR acquisition.
   - **TDR11100** → not a DLL. This instrument is scripted over **gRPC on TCP 50052** using the
     MIT-licensed `radium.v1.Radium` proto at <https://github.com/HYPERLABS/TDR11100>. Use the
     `hyperlabs-capture-tdr-waveform` skill and `grpc/hyperlabs-radium.proto`.
   - **Controlled Impedance Analyzers with IPC coupons** → **Zcoupon**, which publishes no automation
     interface; it is GUI-driven with CSV/PNG/Word report output and batch testing built in.

6. **Read the changelog before recommending an upgrade.** Each package's `resources[]` includes a plain
   text changelog with dated entries classed `FEATURE`, `IMPROVEMENT`, `CHANGE`, `INTERFACE`, `TUNING`,
   `BUGFIX` and `DEPRECATION`. The structured recent window is captured in
   `changelog/hyperlabs-changelog.yml`. Two entries there are genuinely breaking and worth surfacing:
   Zcoupon 2.6.0 (2018-12-17) deprecated support for HL52xx-series instruments, and both XTDR 2.5.0 and
   ZTDR 2.0.1 removed the auto-calibration toggle when multithreading landed.

7. **Check the instrument is still current.** `GET /v1/products/discontinued` lists end-of-life
   products; each product carries `replacementProductId`, so recommend the successor rather than a
   discontinued SKU.

## Rules

- Downloads are served from Azure Blob Storage under
  `https://hyperlabsstprodcus.blob.core.windows.net/hyperlabs/{guid}/{filename}`. Those URLs come from
  the API response — resolve them fresh rather than caching, since the GUID changes when the provider
  republishes an asset.
- `ZTDR DLL 2.1.0` is advertised as version 2.1.0 but the published archive filename is
  `ZTDR_DLL_1.1.2.zip` and the changelog runs only through 2.0.5. Report the discrepancy rather than
  picking one; the DLL manual is the authority.
- All operations in this skill are anonymous GETs. Do not send `Authorization`, and do not escalate into
  the quote or contact-message endpoints — those email HYPERLABS sales and require a `recaptchaToken`.
- Errors, rate-limit behaviour and pagination follow `conventions/hyperlabs-conventions.yml` and
  `errors/hyperlabs-problem-types.yml`.
- HYPERLABS publishes nothing to npm, PyPI, NuGet, Maven Central, RubyGems, Packagist, crates.io or
  pkg.go.dev. Any package on those registries carrying the name is a third party. See
  `packages/hyperlabs-packages.yml`.
