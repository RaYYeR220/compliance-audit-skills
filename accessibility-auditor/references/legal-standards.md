# Accessibility Legal Standards

Loaded by `SKILL.md` Phase 3 to set the legal scope and cite the right regime in findings. All major regimes converge on **WCAG** as the technical benchmark, so the engineering fixes are the same; what differs is *who is legally required to comply* and *what they reference*.

Use the market signals at the end to decide which regimes to cite. When in doubt, target **WCAG 2.2 AA** — it satisfies the strictest common denominator.

---

## European Accessibility Act (EAA) — Directive (EU) 2019/882

- **In force**: enforcement began **June 28, 2025**. This is the most significant recent driver.
- **Who**: private-sector companies selling covered products/services to consumers in the EU — e-commerce, banking, e-books, ticketing, transport, telecoms, computers/OS, smartphones, ATMs/terminals. **Applies regardless of where the company is headquartered** if it sells into the EU.
- **Micro-enterprise exemption**: services from businesses with <10 employees **and** ≤€2M turnover may be exempt (varies by member state and by product vs service).
- **Technical standard**: **EN 301 549**, which currently references **WCAG 2.1 AA** for web/apps. EN 301 549 is being updated to align with WCAG 2.2. So: for EAA today, WCAG 2.1 AA is the operative requirement; treat WCAG 2.2-only criteria (2.4.11, 2.5.7, 2.5.8, 3.2.6, 3.3.7, 3.3.8) as advisory-but-recommended and label them as 2.2 additions.
- **Penalties**: set per member state. Reported ranges €5,000–€500,000; Germany up to ~€100,000 per violation; France €5,000–€250,000 plus periodic penalties for a missing accessibility statement. Non-compliant products can be removed from the EU market.
- **Accessibility statement**: covered services generally must publish one.

**Report stance**: If EU market signals are present, cite EAA + EN 301 549 (WCAG 2.1 AA) and recommend publishing an accessibility statement. Note the 2.2 additions as forward-looking.

---

## Americans with Disabilities Act (ADA) — United States

- **Who**: Title III covers "places of public accommodation" — courts have widely applied this to websites and apps of businesses serving the public. Title II covers state/local government.
- **Standard**: the ADA statute predates the web and does not name a version, but **WCAG 2.1 AA** is the de facto benchmark courts and settlements use. In 2024 the DOJ issued a Title II rule (state/local government) explicitly adopting **WCAG 2.1 AA**, with compliance deadlines in 2026–2027 based on entity size — a strong signal that 2.1 AA is the floor and reinforces the standard broadly.
- **Risk**: high volume of private litigation and demand letters over inaccessible websites/apps, especially e-commerce.
- **Penalties**: civil suits, settlements, injunctive relief, plaintiff attorney fees; federal civil penalties for DOJ actions.

**Report stance**: For US-facing consumer products, cite ADA with WCAG 2.1/2.2 AA as the benchmark. Note the 2024 DOJ Title II rule if the entity is government-related.

---

## Section 508 — US federal

- **Who**: US federal agencies and, in practice, vendors selling ICT to them.
- **Standard**: the 508 Refresh adopts **WCAG 2.0 AA** (via incorporation), commonly satisfied by meeting 2.1 AA.
- **Artifact**: procurement typically requires a **VPAT / Accessibility Conformance Report (ACR)**.

**Report stance**: If the product is sold to US government/enterprise, note that a VPAT/ACR will likely be required and that this static audit is input to it, not a substitute.

---

## EN 301 549 — European harmonized standard

- The technical standard underpinning both the EAA (private sector) and the EU Web Accessibility Directive 2016/2102 (public sector). References WCAG (2.1 AA today; updating toward 2.2) and adds requirements for non-web documents, hardware, and biometrics.

---

## UK

- **Public sector**: Public Sector Bodies Accessibility Regulations 2018 require WCAG 2.1 AA + an accessibility statement.
- **Private sector**: the Equality Act 2010 requires "reasonable adjustments"; WCAG is the practical benchmark. Post-Brexit, UK firms selling into the EU still fall under the EAA for that market.

---

## Canada — AODA and ACA

- **Ontario (AODA)**: requires WCAG 2.0 AA for many organizations' web content.
- **Federal (Accessible Canada Act)**: applies to federally regulated entities; references EN 301 549 / WCAG.

---

## Other regions (brief)

- **Australia**: Disability Discrimination Act; government mandates WCAG 2.1 AA.
- **Israel**: IS 5568 (based on WCAG 2.0 AA), strict enforcement.
- **Japan**: JIS X 8341-3 (aligned to WCAG).
Most map back to WCAG, so the fixes don't change — only the citation does.

---

## What this means for the audit

1. **The engineering target is WCAG 2.2 AA.** Meeting it satisfies essentially every regime above (2.2 is a superset of 2.1, which is a superset of 2.0).
2. **Cite the regime that matches the market** so the user understands the legal weight, but don't change the technical fixes per region.
3. **Version nuance to state explicitly**: EAA/EN 301 549 currently reference **WCAG 2.1 AA**; US DOJ Title II adopts **2.1 AA**; so the 6 WCAG **2.2** additions (2.4.11 Focus Not Obscured, 2.5.7 Dragging, 2.5.8 Target Size, 3.2.6 Consistent Help, 3.3.7 Redundant Entry, 3.3.8 Accessible Authentication) are required under a 2.2 target but only advisory where a regime still cites 2.1. Label these in findings so the user can prioritize.
4. **Accessibility statement / VPAT**: recommend publishing an accessibility statement (EAA/UK public sector) and note that enterprise/government sales will likely need a VPAT/ACR. This audit is input to those, not a substitute.

---

## Market detection signals

- **EU (EAA / EN 301 549)**: locales `de`, `fr`, `es`, `it`, `nl`, `pl`, `en-GB`; currencies EUR/GBP; `.eu`/EU-country domains; mentions of EAA, GDPR, VAT, EU shipping. → cite EAA (in force June 28, 2025), WCAG 2.1 AA, recommend accessibility statement.
- **US (ADA / Section 508)**: `en-US`, USD, `.com`/`.us`, US states/ZIP, government/`.gov` clients. → cite ADA (WCAG 2.1/2.2 AA); Section 508 + VPAT if selling to government/enterprise.
- **Canada (AODA/ACA)**: `en-CA`/`fr-CA`, CAD, Ontario references. → cite AODA (WCAG 2.0 AA) and ACA where federal.
- **Mixed/unclear**: default to "WCAG 2.2 AA — the common denominator across ADA, the EAA, and Section 508," and say so.

If signals are weak, ask the user which markets the product serves before assigning a specific regime, but proceed with WCAG 2.2 AA as the technical target regardless.
