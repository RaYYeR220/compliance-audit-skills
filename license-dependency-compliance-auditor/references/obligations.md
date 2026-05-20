# License Obligations and Compliance Steps

Loaded by `SKILL.md` Phase 3. For each license category, this lists what you must actually **do** to comply, plus the checks to run against the project. Obligations mostly bite **on distribution**; for hosted SaaS, the AGPL/source-available "network" obligations are the ones that matter.

---

## Attribution (almost every open-source license)

**Obligation**: Preserve copyright notices and include the license text when you distribute the software (including binaries, apps, npm packages you publish).

**Applies to**: MIT, BSD, ISC, Apache-2.0, and essentially all permissive and copyleft licenses.

**Check**:
- Are you distributing? (app, binary, mobile app, published package — yes; hosted-only SaaS — attribution still good practice and required for code sent to the browser).
- Is there an aggregated place where third-party licenses are reproduced? Look for `NOTICE`, `THIRD_PARTY_LICENSES`, `licenses.txt`, an in-app "Open Source Licenses" / "Acknowledgements" screen, or a generated attributions file.
- For browser-shipped JS: the minified bundle ships to users; their licenses must be available (many build tools can emit a license file).

**Red flags**:
- Distributing an app/binary with no third-party license file anywhere → **High**.
- Apache-2.0 dependencies present and distributing, but no NOTICE aggregation → **High/Medium**.

**Fix**: Generate and ship an aggregated attributions file or an in-app licenses screen. Tools exist per ecosystem (e.g. license-checker/oss-attribution for npm, pip-licenses for Python, cargo-about for Rust, go-licenses for Go) — recommend, don't run.

---

## Apache-2.0 specifics

**Obligations beyond attribution**:
1. Preserve the `NOTICE` file content if the dependency ships one.
2. State significant changes you made to the files.
3. The patent grant terminates if you bring a patent claim against the project — informational, not an action item.

**Check**: Apache-2.0 dependencies present + distributing → NOTICE aggregated? Modifications to Apache files documented?

**Severity**: Missing NOTICE on distribution → **Medium/High**.

---

## Weak copyleft — LGPL

**Obligation**: Modifications to the LGPL library must be shared under LGPL. Users must be able to replace the library with a modified version ("relinking"):
- **Dynamic linking** (shared lib, separate module): generally satisfies relinking → compatible with proprietary apps.
- **Static linking** into a proprietary binary: you must provide the object files or another mechanism so users can relink → usually impractical for closed apps.

**Check**:
- Is the LGPL component dynamically or statically linked?
- Did you modify it? If so, are the changes published?

**Severity**: Static-linked LGPL in a distributed proprietary binary with no relink mechanism → **High**. Dynamic linking, unmodified → Pass.

**Fix**: Use dynamic linking, or provide the relink mechanism, or replace with a permissive equivalent.

---

## Weak copyleft — MPL-2.0 / EPL

**Obligation**: File-level reciprocity. If you modify an MPL-licensed file, you must make that file's source available under MPL. You may combine MPL files with proprietary files in the same project without licensing your files under MPL.

**Check**: Did you modify any MPL/EPL source files? If yes, are those files' sources available?

**Severity**: Modified MPL files not published while distributing → **High**. Unmodified use → Pass.

**Fix**: Publish the modified MPL/EPL files' source. No need to open your own files.

---

## Strong copyleft — GPL-2.0 / GPL-3.0

**Obligation**: If you **distribute** a work that includes or derives from GPL code, the **entire** combined work must be licensed under the GPL, and you must provide (or offer) the complete corresponding source.

**Check**:
- Distribution model. SaaS (no distribution) → usually not triggered. Distributed app/binary → triggered.
- Is the GPL code a runtime/shipped dependency or a dev/build tool? Running a GPL build tool (e.g. a GPL compiler) does **not** make your output GPL; **linking/embedding** GPL code into your shipped artifact does.

**Severity**:
- GPL library linked/embedded into a **distributed proprietary** artifact → **Critical**.
- GPL **dev/build tool**, output not derived → Pass (note as Low).
- GPL in **SaaS** backend, not distributed → generally Pass, but note that converting to an on-prem/distributed offering later would trigger it (Medium advisory).

**Fix**: Replace the GPL runtime dependency with a permissive equivalent, isolate it as a separate process invoked at arm's length (aggregation, not derivation — fact-specific, route to review), or release your work under GPL.

---

## Strong network copyleft — AGPL-3.0

**Obligation**: Everything in GPL-3.0, **plus** Section 13: if users interact with the software **over a network**, you must offer them the **complete corresponding source** of your version. There is no "SaaS loophole".

**Check**:
- Is an AGPL component part of a hosted/network service? → triggered.
- Is the surrounding application proprietary? → conflict.

**Severity**: AGPL component in proprietary SaaS/hosted service → **Critical**. (This is the most common high-impact finding for web startups.)

**Fix**: Remove/replace the AGPL component with a permissive alternative; obtain a commercial license from the vendor (many AGPL projects dual-license commercially); or open-source your application under AGPL. Isolating AGPL behind a network/process boundary is sometimes argued but is fact-specific — route to legal review, do not assert it's safe.

---

## Source-available (BUSL / SSPL / Elastic-2.0 / FSL / Confluent / RSAL)

**Obligation**: Use only within the license's grant. The common restriction is: **you may not offer the software (or a product based on it) as a commercial/hosted service that competes with the licensor**. Some (BUSL, FSL) convert to open licenses after a change date.

**Check**:
- Does your use fall inside the grant (internal use, non-competing application) or outside (you host/resell the component's functionality)?
- For BUSL/FSL, is the dependency version past its change date (then it's under the converted open license)?
- SSPL specifically: offering the software as a service requires open-sourcing the entire service stack — effectively prohibitive for proprietary SaaS.

**Severity**:
- Using a source-available component in a way the license forbids (e.g. hosting it commercially) → **Critical**.
- Use within grant but commercial product → **Medium** (surface so the user confirms the grant applies).
- Version already converted to open license → Pass (note the converted license).

**Fix**: Confirm your use is within the grant in writing; obtain a commercial license; switch to an open fork (e.g. OpenTofu for Terraform, OpenSearch for older Elasticsearch, Valkey for Redis); or wait for/upgrade to a version under the converted/open license.

---

## The project's own license

**Check**:
- Does the repo have a `LICENSE`/`COPYING` file and a `license` field in the manifest, and do they agree?
- If the project is open source, is its outbound license **compatible** with all inbound dependency licenses? (e.g. an MIT project bundling GPL code and distributing must actually be GPL.)
- If proprietary, is the `license` field set to `UNLICENSED`/`proprietary` (npm) or marked private, so it isn't accidentally published as open?

**Severity**:
- Distributing a product that bundles licensed code while the project itself has **no** license declared → **High** (ambiguous rights for recipients; possible inbound conflicts hidden).
- Manifest `license` contradicts the LICENSE file → **Medium**.
- Open-source project whose declared license is incompatible with a bundled dependency → **High/Critical** depending on the dependency.

**Fix**: Add a clear LICENSE consistent with the distribution model and compatible with all dependencies. For proprietary projects, set the manifest license to `UNLICENSED`/`proprietary` and `private: true` where applicable.

---

## Vendored / copied code

**Check**: Code copied into `vendor/`, `third_party/`, `libs/`, or single files pasted from other projects. These ship directly with your code and carry their original license and attribution duties, but are invisible to package-manager scanning.

**Severity**: Vendored copyleft code in a distributed proprietary product → as per its license (often **Critical**). Vendored permissive code with stripped headers → **High** (attribution removed).

**Fix**: Restore original license headers; track vendored components in a manifest; replace copyleft vendored code if incompatible.

---

## Dependency with no resolvable license

**Check**: Lock file or metadata yields no license, `UNLICENSED`, empty, or unresolvable.

**Severity**: Shipping/depending on code with no license grant → **Critical/High** (no legal right to use it).

**Fix**: Find the upstream license; if none exists, the code is all-rights-reserved — contact the author for terms or remove it.

---

## Compliance summary the report should drive toward

1. Every distributed dependency's attribution is reproduced somewhere users can access.
2. No copyleft conflict given the distribution model (no AGPL in proprietary SaaS; no GPL in proprietary distributed binaries).
3. No source-available component used outside its grant.
4. No dependency with an unknown/absent license shipping.
5. The project's own license exists, is internally consistent, and is compatible with everything it bundles.
