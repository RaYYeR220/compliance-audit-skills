# License & Dependency Compliance Audit

**Generated**: 2026-05-20
**Ecosystems**: npm (pnpm-lock.yaml), Go modules
**Project license**: none found (no LICENSE file; `package.json` "license" field absent)
**Assumed distribution model**: Proprietary SaaS (hosted) — Dockerfile + Next.js web app + Go API backend, `private: true` in package.json, no publish config
**Dependencies analyzed**: 47 direct, 612 transitive (dev-only excluded from Critical/High; browser-shipped frontend deps treated as distributed)

## Summary

| Severity | Count |
|----------|-------|
| Critical | 2     |
| High     | 3     |
| Medium   | 3     |
| Low      | 2     |
| **Total**| 10    |

## License distribution

| License | SPDX | Count | Category |
|---------|------|-------|----------|
| MIT | `MIT` | 401 | permissive |
| ISC | `ISC` | 78 | permissive |
| Apache-2.0 | `Apache-2.0` | 96 | permissive (NOTICE duty) |
| BSD-3-Clause | `BSD-3-Clause` | 31 | permissive |
| MPL-2.0 | `MPL-2.0` | 3 | weak-copyleft |
| AGPL-3.0 | `AGPL-3.0-only` | 1 | strong-copyleft |
| BUSL-1.1 | `BUSL-1.1` | 1 | source-available |
| none/unknown | — | 1 | unknown |

## Critical

### C-1. `rrweb@2.0.0` is AGPL-3.0, bundled into a proprietary hosted product
- **Dependency**: `rrweb@2.0.0` (`pnpm-lock.yaml`; imported in `apps/web/src/lib/replay.ts:3`)
- **License**: AGPL-3.0-only (strong network copyleft)
- **Conflict**: The product ships as hosted SaaS. AGPL §13 treats providing functionality over a network as distribution, so embedding AGPL code obligates you to offer the **complete corresponding source of the whole application** to your users under AGPL. That is incompatible with keeping the app proprietary.
- **Obligation or risk**: Either release the full app under AGPL, or stop using the AGPL component. As shipped today, the product is out of compliance.
- **Fix**: Replace with a permissively licensed session-replay library (MIT/Apache alternative) or obtain a commercial license from the vendor if one is offered. Isolating it behind a process boundary is sometimes argued but is fact-specific — get legal review before relying on it.

### C-2. `fast-geoip-pro@1.4.2` ships with no license
- **Dependency**: `fast-geoip-pro@1.4.2` (`pnpm-lock.yaml`; used in `apps/api/middleware/geo.go` via a vendored copy under `third_party/geoip/`)
- **License**: none found — no LICENSE file, empty `license` field, no SPDX header
- **Conflict**: Without a license grant, the default is **all rights reserved**. You have no legal right to use, copy, or distribute it, in any model.
- **Obligation or risk**: Using unlicensed code in production is copyright infringement exposure.
- **Fix**: Locate the upstream license; if none exists, contact the author for terms or remove the dependency. The vendored copy under `third_party/geoip/` also strips any original notices — restore them if a license is found.

## High

### H-1. Project has no LICENSE while bundling third-party code
- **Dependency**: the project itself (`/` — no `LICENSE`, no `license` field in `package.json`)
- **License**: undeclared
- **Conflict**: A proprietary product with no declared license is ambiguous to recipients and to auditors (e.g. in due diligence). It also hides inbound conflicts. For npm, an absent license can be misread as unlicensed.
- **Fix**: Add an explicit proprietary license / set `package.json` `"license": "UNLICENSED"` and `"private": true`. Document internal licensing. This also unblocks a clean compliance story for fundraising/acquisition.

### H-2. Apache-2.0 dependencies shipped to the browser with no aggregated NOTICE
- **Dependency**: 96 Apache-2.0 packages (e.g. `@opentelemetry/api@1.9.0`, `protobufjs@7.2.6`) bundled into the client-side JS in `apps/web`
- **License**: Apache-2.0 (permissive, but requires preserving NOTICE content on distribution)
- **Conflict**: Even though the product is "SaaS", the frontend bundle is distributed to users' browsers. Apache-2.0 requires preserving applicable NOTICE files; no aggregated notices were found.
- **Fix**: Generate an aggregated third-party attributions file (e.g. an "Open Source Licenses" page) covering all bundled licenses, including the Apache NOTICE contents. Recommended tooling: an attribution generator in your build (do not run automatically — add it to the pipeline).

### H-3. `terraform-provider-internal@1.2.0` is BUSL-1.1
- **Dependency**: `terraform-provider-internal@1.2.0` (`go.mod`; used in `infra/`)
- **License**: BUSL-1.1 (source-available)
- **Conflict**: BUSL forbids using the software to offer a competing commercial/hosted service before the change date. If this provider is only used for your own internal infrastructure, that is typically within the Additional Use Grant; if any part is exposed as a service, it is not.
- **Fix**: Confirm in writing that your use falls within the grant (internal infra only). If the version is past its change date it converts to an open license — check the date. Otherwise consider OpenTofu/an open provider. Currently classed High pending confirmation of use.

## Medium

### M-1. Dual-licensed dependencies — chosen option not documented
- **Dependency**: 14 packages under `MIT OR Apache-2.0` (common in Rust/JS toolchains)
- **License**: dual (you may pick either)
- **Fix**: Not a violation. Document your chosen option (typically MIT for simplicity) in your compliance notes so attribution is consistent.

### M-2. MPL-2.0 dependencies — confirm files unmodified
- **Dependency**: `certifi`-style + 2 others under MPL-2.0 (`pnpm-lock.yaml`)
- **License**: MPL-2.0 (file-level weak copyleft)
- **Fix**: MPL is fine in proprietary products **if** you don't modify the MPL files (or you publish those modifications). Confirm no patched copies exist in `node_modules` overrides/patches.

### M-3. `go.sum` present but Go license scan limited to module files
- **Dependency**: Go module set
- **Fix**: Go modules don't carry structured license fields; licenses were read from each module's LICENSE file. Two modules had non-standard license filenames — listed under "Needs human review".

## Low

### L-1. No SPDX headers in first-party source
- **Location**: `apps/web/src`, `apps/api`
- **Fix**: Add SPDX headers (`// SPDX-License-Identifier: UNLICENSED`) to first-party files for clarity in audits.

### L-2. License metadata not pinned in CI
- **Fix**: Add a CI step that fails the build on a disallowed license category (e.g. any AGPL/BUSL/SSPL/unknown in runtime deps). This prevents regressions like C-1 from reappearing.

## Attribution checklist (browser-shipped frontend)

Reproduce license text + copyright for these categories in an in-app "Open Source Licenses" page: MIT (401), ISC (78), Apache-2.0 (96 — include NOTICE), BSD-3-Clause (31), MPL-2.0 (3). The AGPL component (C-1) must be resolved, not merely attributed.

## Passed / low-risk

- 401 MIT, 78 ISC, 31 BSD-3-Clause dependencies — permissive, compatible with proprietary SaaS; only attribution (for browser-shipped) applies.
- Dev-only dependencies (eslint, vitest, types) — not shipped; copyleft among them carries no distribution obligation here.

## Needs human review

- `fast-geoip-pro` (C-2) — license undeterminable; decide use or removal.
- 2 Go modules with non-standard license filenames — read and classify manually.
- `terraform-provider-internal` BUSL grant (H-3) — confirm internal-only use in writing.

## Next steps

1. **Remove or relicense `rrweb` (C-1)** — the only finding that makes the product non-compliant as it ships. Pick a permissive replacement this sprint.
2. **Resolve the unlicensed `fast-geoip-pro` (C-2)** — find the license or remove it.
3. **Add a project LICENSE / `UNLICENSED` + `private` (H-1).**
4. **Ship an aggregated attributions page (H-2)** before the next release.
5. **Add the CI license gate (L-2)** to prevent regressions.

---

*This audit is an engineering analysis of declared licenses, not legal counsel. License interpretation is fact-specific; consult an attorney for binding decisions, especially before a release, fundraise, or acquisition.*
