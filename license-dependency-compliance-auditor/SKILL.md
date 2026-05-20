---
name: license-dependency-compliance-auditor
description: Audits a project's open-source dependencies for license compliance risk — copyleft contamination, attribution duties, and source-available restrictions that conflict with how the code ships. Detects the package ecosystems in use (npm, PyPI, Cargo, Go modules, Maven/Gradle, RubyGems, Composer, NuGet, pub.dev), reads the declared license of each direct and transitive dependency, infers how the project is distributed (proprietary SaaS, distributed binary, open source, internal tool), and flags conflicts. Catches AGPL in closed SaaS, GPL in proprietary distributed software, missing attribution/NOTICE files, source-available licenses (BUSL, SSPL, Elastic License, FSL) used commercially against their terms, dependencies with no license (all-rights-reserved), and a missing or mismatched license on the project itself. Produces a single LICENSE_AUDIT.md report graded by legal risk (Critical/High/Medium/Low) with the dependency, its license, the SPDX identifier, the obligation or conflict, and a concrete remediation. Use this skill when the user asks to audit licenses, check dependency licenses, review open-source compliance, check for GPL or copyleft, verify attribution requirements, prepare for due diligence or acquisition, check if a license is compatible, review the license of the project, audit third-party code, or check for source-available license risk before shipping.
---

# License & Dependency Compliance Auditor

You are auditing a project's software supply chain for open-source license risk. Your goal is to find license obligations and conflicts that create legal exposure given how the project actually ships, and produce a clear, prioritized report.

## Important boundary

This skill produces an engineering license audit, not legal advice. End every report with: *"This audit is an engineering analysis of declared licenses, not legal counsel. License interpretation is fact-specific; consult an attorney for binding decisions, especially before a release, fundraise, or acquisition."*

License conflicts depend on facts the code cannot fully reveal (how you distribute, whether you modify a library, your commercial model). When a verdict depends on such facts, state the assumption and flag the ambiguity rather than asserting.

## Workflow

Execute these phases in order.

### Phase 1 — Detect ecosystems and distribution model

1. Identify package ecosystems from manifest and lock files:
   - **npm / Node**: `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`.
   - **Python**: `pyproject.toml`, `requirements*.txt`, `Pipfile.lock`, `poetry.lock`, `setup.py`/`setup.cfg`.
   - **Rust**: `Cargo.toml`, `Cargo.lock`.
   - **Go**: `go.mod`, `go.sum`.
   - **Java/Kotlin**: `pom.xml`, `build.gradle(.kts)`.
   - **Ruby**: `Gemfile`, `Gemfile.lock`.
   - **PHP**: `composer.json`, `composer.lock`.
   - **.NET**: `*.csproj`, `packages.config`, `*.sln`.
   - **Dart/Flutter**: `pubspec.yaml`, `pubspec.lock`.
   Load `references/ecosystems.md` for where each stores license metadata.
2. Determine the project's own license: look for `LICENSE`, `LICENSE.md`, `COPYING`, the `license` field in the manifest, or SPDX headers. Record it, or note its absence.
3. Infer the **distribution model** — this is the single most important factor for copyleft risk:
   - **Proprietary SaaS / hosted**: server-side code not distributed to users (web app, API backend). AGPL is the key risk here; GPL/LGPL usually are not triggered because there's no "distribution".
   - **Distributed binary / app**: desktop app, mobile app, CLI, on-prem software, npm package shipped to others. GPL/LGPL copyleft is triggered by distribution.
   - **Open source**: the project itself is under an OSI license. Focus shifts to inbound/outbound compatibility.
   - **Internal tool**: not distributed externally. Lowest copyleft risk, but attribution and source-available terms can still apply.
   Signals: presence of a Dockerfile + web framework (SaaS); `bin`/`main` building a binary or an Electron/mobile config (distributed); `private: true` in package.json or no publish config (internal); a permissive/copyleft LICENSE on the repo (open source).
   If the model is ambiguous, ask the user one short question: *"How does this ship — hosted SaaS, a distributed app/binary, an open-source project, or an internal tool? It changes which license obligations apply."*

State the detected ecosystems, project license, and assumed distribution model before Phase 2.

### Phase 2 — Discovery: build the dependency license map

For each detected ecosystem, enumerate dependencies and their declared licenses.

1. Prefer the **lock file** for the full transitive set; fall back to the manifest for direct dependencies if no lock file exists (note the limitation — transitive deps unseen).
2. For each dependency, determine the SPDX license identifier. Sources, in order of reliability:
   - The license field in the installed package metadata (`node_modules/<pkg>/package.json` `license`; Python `*.dist-info/METADATA`; Rust crate metadata; etc.). See `references/ecosystems.md`.
   - The manifest/registry-declared license.
   - A `LICENSE`/`COPYING` file inside the package.
3. Flag dependencies where the license is:
   - **Missing / `UNLICENSED` / "SEE LICENSE IN..." / empty** — treat as all-rights-reserved until proven otherwise.
   - **A SPDX expression with `OR`/`AND`** — note the choice; dual licenses often let you pick the permissive option.
   - **Non-standard or custom** — record the raw string for manual review.
4. Note any dependency that is **vendored** (copied into the repo under `vendor/`, `third_party/`, `libs/`) — these ship with your code directly and carry attribution duties even if not in a package manager.
5. Distinguish **dev/test/build-only** dependencies from **runtime/shipped** dependencies. A GPL build tool you run but don't distribute is usually fine; a GPL library you link and ship is not. Most ecosystems mark dev dependencies (`devDependencies`, `[dev-dependencies]`, `test`/`provided` scope) — use that.

Build the map internally. Do not output it yet.

### Phase 3 — Audit

Load `references/license-types.md` (categories, SPDX, compatibility) and `references/obligations.md` (per-license duties). Apply them against the dependency map and the distribution model.

For each dependency or group, evaluate:
- **Compatibility** with the project's own license and distribution model (e.g. AGPL dependency in proprietary SaaS = conflict).
- **Obligations** triggered (attribution, NOTICE file, source disclosure, copyleft reciprocity, patent/notice terms).
- **Source-available restrictions** (BUSL/SSPL/ELv2/FSL/Confluent) that may prohibit your commercial use.
- **Unknown/missing licenses** that block safe use.

Record pass / conflict / obligation-unmet / needs-review per item, with severity.

**Severity = legal risk** (use consistently):

- **Critical** — High likelihood of a license violation as the project ships today. Examples: AGPL dependency embedded in proprietary SaaS; GPL library linked into a distributed proprietary binary; a source-available (BUSL/SSPL) component used in a way its license forbids; shipping a dependency with no license at all.
- **High** — Clear unmet obligation or a strong conflict requiring action before release. Examples: copyleft (LGPL/MPL) used without meeting its conditions; required attribution/NOTICE entirely absent while distributing; project distributed with no LICENSE while bundling licensed code.
- **Medium** — Obligation partially met or risk depends on facts. Examples: attribution present but incomplete; permissive licenses with NOTICE requirements not aggregated; dual-licensed dependency where the safe option isn't documented.
- **Low** — Hygiene. Examples: project missing its own LICENSE while internal-only; SPDX headers absent; license metadata not pinned.

### Phase 4 — Report

Write `LICENSE_AUDIT.md` at the repository root (mention overwrite before writing). Use this structure:

```markdown
# License & Dependency Compliance Audit

**Generated**: <ISO date>
**Ecosystems**: <e.g. "npm (pnpm-lock), Go modules">
**Project license**: <e.g. "none found" / "MIT" / "proprietary">
**Assumed distribution model**: <SaaS | distributed app | open source | internal> — <why>
**Dependencies analyzed**: <N direct, M transitive> (<dev-only excluded / included>)

## Summary

| Severity | Count |
|----------|-------|
| Critical | N     |
| High     | N     |
| Medium   | N     |
| Low      | N     |
| **Total**| N     |

## License distribution

| License | SPDX | Count | Category |
|---------|------|-------|----------|
| ...     | ...  | ...   | permissive / copyleft / weak-copyleft / source-available / unknown |

## Critical

### C-1. <Dependency> — <conflict in one line>
- **Dependency**: `name@version` (`path or manifest:line`)
- **License**: <SPDX id and category>
- **Conflict**: <why it conflicts with the distribution model / project license>
- **Obligation or risk**: <what the license requires or forbids>
- **Fix**: <replace with X (permissive alt), or comply by Y, or get a commercial license>

## High
### H-1. ...

## Medium
### M-1. ...

## Low
### L-1. ...

## Attribution checklist
If distributing: the licenses below require their text and copyright notice to be reproduced (e.g. in an in-app "Licenses" screen, a NOTICE file, or docs). List the dependencies whose notices must be included.

## Passed / low-risk
Terse: permissive dependencies (MIT/BSD/Apache/ISC) with obligations met.

## Needs human review
Dependencies with custom, unknown, or dual licenses where a person must decide.

## Next steps
Priority order, Criticals first.

---

*This audit is an engineering analysis of declared licenses, not legal counsel. License interpretation is fact-specific; consult an attorney for binding decisions, especially before a release, fundraise, or acquisition.*
```

Rules:
- Always name the dependency and version and where it's declared.
- State the SPDX identifier and category for every finding.
- Tie each conflict to the **distribution model** — the same dependency can be fine in one model and a violation in another. Make the assumption explicit.
- Fixes must be concrete: a named permissive replacement, the specific compliance step, or "obtain a commercial license from <vendor>".
- Do not assert a violation when it depends on unrevealed facts; route those to "Needs human review" with the question to answer.
- Never invent a license. If you cannot determine it, mark it unknown and list it for review.

### Phase 5 — Summary to user

After writing, give a 5-7 line summary: where the report is, counts by severity, the single most dangerous Critical (with the dependency name), the recommended first action, and the not-legal-advice reminder. Don't paste the full report.

## Output format example

A good Critical finding:

```markdown
### C-1. `rrweb@2.0.0` is bundled in a proprietary SaaS but is AGPL-3.0
- **Dependency**: `rrweb@2.0.0` (`pnpm-lock.yaml`; imported in `apps/web/src/session-replay.ts:4`)
- **License**: AGPL-3.0-only (strong network copyleft)
- **Conflict**: The product ships as hosted SaaS. AGPL's section 13 treats network interaction as distribution, so using AGPL code in a hosted service obligates you to offer the **complete corresponding source of your application** to your users under AGPL. That is incompatible with keeping the app proprietary.
- **Obligation or risk**: Either release your full app source under AGPL, or stop using the AGPL component.
- **Fix**: Replace with a permissively licensed session-replay library (e.g. an MIT/Apache alternative) or obtain a commercial license from the vendor if offered. Until then, treat the product as out of compliance.
```

A weak finding to avoid:

```markdown
### Bad — do not do this
- Dependency: some GPL thing
- License: GPL
- Fix: remove GPL stuff
```

## Edge cases and rules of thumb

- **No lock file**: Audit direct dependencies from the manifest; clearly state transitive dependencies were not analyzed and recommend generating a lock file.
- **Monorepo / multiple ecosystems**: Audit each package/ecosystem; attribute findings to the package path.
- **Dev vs runtime**: Default to focusing on shipped/runtime dependencies. Note dev-only copyleft separately as Low unless the build output embeds it.
- **Dual licensing (`MIT OR Apache-2.0`)**: Not a problem — you may choose the option that suits you. Document the chosen option. Flag only if neither option is compatible.
- **LGPL nuance**: Dynamic linking + ability for users to relink is usually compatible even in proprietary distribution; static linking without that ability is not. Note the linking mode if determinable; otherwise route to review.
- **Permissive with NOTICE (Apache-2.0)**: Requires preserving the NOTICE file content on distribution — a common miss. Flag if distributing and no aggregated notices exist.
- **Source-available whipsaw**: Some projects changed licenses across versions (e.g. Redis, Elasticsearch, Terraform). Pin findings to the **version in the lock file**, not the project's latest license.
- **Your own code copied from Stack Overflow / snippets**: out of scope for dependency scanning; mention only if obvious license headers appear in vendored files.
- **Existing audit file**: If `LICENSE_AUDIT.md` exists, ask whether to overwrite or append a dated section.

## What this skill does NOT do

- Does not modify code, swap dependencies, or generate NOTICE files automatically — it reports and recommends.
- Does not provide legal advice or a definitive compatibility ruling.
- Does not scan for security vulnerabilities (use a dedicated SCA/audit tool for CVEs).
- Does not detect license text plagiarized into source without headers, or verify that a declared license matches the actual code.
- Does not check patent grants or trademark terms beyond what the license states.
