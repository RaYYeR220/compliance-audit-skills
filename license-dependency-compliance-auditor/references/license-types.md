# License Types, SPDX, and Compatibility

Loaded by `SKILL.md` Phase 3. Use this to categorize each dependency's license and judge compatibility with the project's distribution model. SPDX identifiers are the canonical short names (see spdx.org/licenses).

The decisive question is almost always: **how does the project ship?** A license that is harmless in hosted SaaS can be a violation in a distributed binary, and vice versa.

---

## Category 1 — Permissive

Minimal obligations: keep the copyright notice and license text. No requirement to release your source. Safe in virtually all distribution models.

| License | SPDX | Notes |
|---------|------|-------|
| MIT | `MIT` | Most common. Keep copyright + license text. |
| ISC | `ISC` | Functionally like MIT. |
| BSD 2-Clause | `BSD-2-Clause` | Keep notice. |
| BSD 3-Clause | `BSD-3-Clause` | Plus no-endorsement clause. |
| Apache-2.0 | `Apache-2.0` | Permissive **plus**: explicit patent grant, and you must preserve the `NOTICE` file content on distribution. State changes must be marked. |
| Unlicense / 0BSD / CC0-1.0 | `Unlicense` / `0BSD` / `CC0-1.0` | Public-domain-equivalent. No obligations. |
| Python-2.0, Zlib, BSL-1.0 (Boost) | `Zlib`, `BSL-1.0` | Permissive. (Note: Boost `BSL-1.0` is permissive — do not confuse with Business Source License "BUSL".) |

**Audit stance**: Pass, provided attribution is preserved when distributing. Apache-2.0's NOTICE requirement is the common miss.

---

## Category 2 — Weak copyleft (file/library-level reciprocity)

You must share modifications **to the licensed files/library**, but you can combine them with proprietary code under conditions. Usually compatible with proprietary distribution if you don't modify the library and (for LGPL) allow relinking.

| License | SPDX | Trigger / condition |
|---------|------|--------------------|
| LGPL-2.1 / LGPL-3.0 | `LGPL-2.1-only`, `LGPL-3.0-only` | If you modify the library, share those changes. Proprietary apps must allow users to **relink** a modified library — easy with dynamic linking, hard with static linking. |
| MPL-2.0 | `MPL-2.0` | File-level copyleft: modified MPL files must be shared, but you may combine with proprietary files in the same project. Common and business-friendly. |
| EPL-2.0 | `EPL-2.0` | Eclipse; module-level reciprocity. |
| CDDL-1.0 | `CDDL-1.0` | File-level; historically GPL-incompatible. |

**Audit stance**: Usually OK to use in proprietary software **if** you don't modify the licensed files (or you publish those modifications) and, for LGPL, you preserve relinking. Static linking of LGPL into a proprietary binary without relinking ability → route to review or flag High.

---

## Category 3 — Strong copyleft (whole-work reciprocity on distribution)

If you **distribute** a work that includes this code, the entire combined work must be offered under the same license, with complete corresponding source. Fundamentally incompatible with proprietary **distributed** software.

| License | SPDX | Trigger |
|---------|------|---------|
| GPL-2.0 | `GPL-2.0-only` / `GPL-2.0-or-later` | Distribution of a derived/combined work → whole work under GPL-2.0. |
| GPL-3.0 | `GPL-3.0-only` / `GPL-3.0-or-later` | As GPL-2.0, plus patent and anti-tivoization terms. |
| AGPL-3.0 | `AGPL-3.0-only` / `AGPL-3.0-or-later` | **Section 13**: providing the software's functionality **over a network** counts like distribution. Using AGPL in hosted SaaS obligates offering your application's complete source to users. This is the #1 trap for SaaS. |

**Audit stance by distribution model**:
- **Proprietary SaaS**: GPL/LGPL generally **not** triggered (no distribution to users). **AGPL is triggered** → Critical if the app is proprietary.
- **Distributed proprietary binary/app**: GPL and AGPL both triggered → Critical. LGPL conditional (see Category 2).
- **Open-source project**: depends on outbound license compatibility (below).
- **Internal-only**: copyleft generally not triggered (no distribution), but plan ahead if that may change.

---

## Category 4 — Source-available / "fair source" (NOT open source)

Source is visible but use is restricted — typically you may not offer the software as a competing commercial/hosted service, or use is time-limited. These are **not** OSI-approved. Treat commercial use against their terms as a Critical risk.

| License | SPDX (where defined) | Restriction |
|---------|----------------------|-------------|
| Business Source License 1.1 | `BUSL-1.1` | Free use except the "Additional Use Grant" exclusions (often: no offering it as a hosted service). Converts to an open license (often Apache/GPL) after a change date (commonly 4 years). Used by HashiCorp (Terraform/Vault/Consul, 2023), MariaDB MaxScale, CockroachDB, Sentry (earlier). |
| Server Side Public License | `SSPL-1.0` | Like AGPL but stronger: offering the software as a service requires open-sourcing the **entire service stack**. Used by MongoDB (2018), and by Elastic/Redis during their SSPL periods. |
| Elastic License 2.0 | `Elastic-2.0` | No providing the product as a managed service, no circumventing license keys, no removing notices. |
| Functional Source License | `FSL-1.1-MIT` / `FSL-1.1-ALv2` | Sentry's license: no competing use; converts to MIT/Apache after 2 years. |
| Confluent Community License | (custom) | No offering as a SaaS competing with the licensor. |
| Redis RSALv2 | (custom) | Redis Source Available License; no competing managed service. |

**Version whipsaw — important**: Several projects changed licenses across versions. Pin your finding to the **version in the lock file**, not the project's current license:
- **Elasticsearch/Kibana**: Apache-2.0 → SSPL/Elastic-2.0 (2021) → **AGPL added back (Sept 2024)**.
- **Redis**: BSD → RSALv2/SSPL (March 2024) → **AGPL-3.0 in Redis 8 (May 2025)**.
- **Terraform/Vault/Consul**: MPL-2.0 → **BUSL-1.1 (2023)**. OpenTofu forked from the last MPL Terraform.

**Audit stance**: If a dependency under one of these is used in a way the license restricts (most commonly: you run a hosted/commercial service and the license forbids that), it's **Critical**. If your use clearly falls within the grant (internal use, non-competing), note it but still surface it as Medium so the user confirms. Always check the version's actual license.

---

## Category 5 — Unknown / none / proprietary

| Situation | Meaning |
|-----------|---------|
| No license file, no metadata, `UNLICENSED` | **All rights reserved** by default. You have no right to use, copy, or distribute it. Treat as Critical/High until clarified. |
| `"SEE LICENSE IN <file>"` | Custom terms — read the file; route to review. |
| Proprietary / commercial | Used under a separate commercial agreement — confirm the agreement covers your use. |

---

## Compatibility quick-reference (inbound dependency → your project)

"Can I include a dependency under license X in my project under license Y / model Z?"

| Dependency license | Permissive project (MIT/Apache) | Proprietary **distributed** | Proprietary **SaaS** | GPL project |
|--------------------|:---:|:---:|:---:|:---:|
| MIT / BSD / ISC | ✅ | ✅ | ✅ | ✅ |
| Apache-2.0 | ✅ | ✅ (keep NOTICE) | ✅ (keep NOTICE) | ✅ GPL-3.0 only (not GPL-2.0-only) |
| MPL-2.0 | ✅ | ✅ (share modified MPL files) | ✅ | ✅ |
| LGPL | ✅ | ⚠️ relinking required | ✅ | ✅ |
| GPL-2.0/3.0 | ⚠️ makes your distributed work GPL | ❌ | ✅ (not distributed) | ✅ |
| AGPL-3.0 | ⚠️ makes your work AGPL | ❌ | ❌ | ✅ (AGPL/GPL-3.0) |
| BUSL/SSPL/Elastic/FSL | ❌ if you offer competing service | ⚠️ per grant | ❌ if hosting is restricted | ❌ generally |
| None / unknown | ❌ | ❌ | ❌ | ❌ |

Legend: ✅ generally fine · ⚠️ conditional, state the assumption / route to review · ❌ conflict, flag.

Notable nuance: **GPL-2.0-only is incompatible with Apache-2.0** (patent clause); GPL-3.0 is compatible with Apache-2.0. When a project mixes GPL-2.0-only and Apache-2.0 code and distributes, flag it.

---

## How to assign SPDX category in the report

When you record a license, map it to one of: `permissive`, `weak-copyleft`, `strong-copyleft`, `source-available`, `unknown/proprietary`. Use the SPDX id where one exists; for custom licenses, record the raw string and category `unknown/proprietary`.
