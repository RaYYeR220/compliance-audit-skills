# Extended Jurisdictions: LGPD, PIPL, DPDPA

This file is loaded by `SKILL.md` during Phase 3 **only when** one of these jurisdictions was detected in Phase 1 or confirmed by the user. Each section lists only the engineering-actionable differences from GDPR. If a control is identical to GDPR, do not repeat it — note "see GDPR Art. X" in the report.

If none of these are in scope, do not load this file.

**Severity guidance for findings from this file**: When the user has confirmed they target the jurisdiction in question, use the same severity scale as the other checklists: Critical when there is risk of regulatory fine or active enforcement (e.g. cross-border transfer with no mechanism, data localization breach for CIIO scope, ad-tech on under-18 Indian users), High when the law is clearly engaged and a specific obligation is unmet, Medium when partial compliance exists but a specific gap remains, Low when the gap is process or documentation rather than a substantive control. When the jurisdiction was only inferred from weak signals, downgrade by one level and note the inference in the finding.

---

# LGPD — Brazil (Lei Geral de Proteção de Dados, Lei nº 13.709/2018)

Effective since September 2020, enforced by ANPD. Largely modeled on GDPR. The core controls (rights, lawful basis, security, breach notification, processor obligations) closely mirror GDPR and any GDPR-compliant implementation usually satisfies LGPD with minor delta.

## L1. Lawful basis — Art. 7

**Difference**: LGPD recognizes **10 lawful bases** (vs GDPR's 6), adding:
- Studies by research entities (anonymization required where possible).
- Regular exercise of rights in judicial, administrative, or arbitration proceedings.
- Health protection.
- Credit protection (only this basis, not "legitimate interest", supports credit scoring activities).

**Engineering action**: For Brazilian users, ensure the lawful basis claim aligns with LGPD's enumerated bases. Credit-related products must explicitly cite Art. 7, XI.

## L2. Data Protection Officer — "Encarregado"

**Difference**: LGPD requires a designated **Encarregado** (data protection officer) for all controllers, with limited exemptions (small-scale processing, certain micro-enterprises). The threshold is lower than GDPR.

**Engineering action**: Verify with operations that an Encarregado is named and contactable. Their contact must appear in the privacy policy.

## L3. Children — Art. 14

**Difference**: LGPD defines a child as under **12**. Specific parental consent rules apply to processing of children's data; "best interests of the child" is the guiding principle.

**Engineering action**: If targeting users in Brazil, use 12 as the age threshold for parental consent flows (vs 13 for COPPA, 16/13 for GDPR).

## L4. Cross-border transfers — Art. 33

**Difference**: International transfers are permitted only under specific conditions: adequacy decision by ANPD, contractual clauses approved by ANPD, certifications, or specific consent. Adequacy decisions are still emerging.

**Engineering action**: For data flowing out of Brazil to US-hosted services (very common), document the lawful basis for transfer. ANPD has been publishing model clauses — check ANPD resolutions for the current state.

## L5. ANPD breach notification

**Difference**: Breach notification is to the **ANPD** within a "reasonable timeframe" (ANPD has interpreted this as up to 3 working days for high-risk breaches via Resolução CD/ANPD No. 15/2024).

**Engineering action**: Update incident-response runbook to include ANPD contact and the 3-working-day expectation alongside the GDPR 72-hour timeline.

## L6. Anonymization — Art. 12

**Difference**: LGPD explicitly recognizes anonymization as removing the data from scope. Pseudonymized data remains in scope.

**Engineering action**: When anonymizing for deletion-cascade (e.g. invoices kept for tax law), document that the irreversibility criterion is met. Confirm the technique (k-anonymity, tokenization with destroyed mapping) actually prevents re-identification.

---

# PIPL — China (Personal Information Protection Law, 2021)

Effective since November 2021, enforced by the Cyberspace Administration of China (CAC). Strict, with extraterritorial reach when targeting Chinese consumers or analyzing their behavior.

## P1. Separate consent per purpose — Art. 14, 23, 25, 26, 29, 39

**Difference**: PIPL frequently requires **separate, specific consent** for distinct activities: sharing with third parties, cross-border transfers, processing sensitive personal information, automated decision-making, processing of children's data. A single combined "I agree" is insufficient.

**Engineering action**: For Chinese users, decompose consent flows into discrete toggles per activity. Record each consent separately with timestamp and version.

## P2. Sensitive personal information — Art. 28

**Difference**: PIPL's sensitive-PI category is broad: biometrics, religious beliefs, specific identity, medical health, financial accounts, **location tracking (including precise location)**, and any PI of children under 14. Explicit consent and a documented necessity assessment are required.

**Engineering action**: Treat precise geolocation as sensitive PI for Chinese users (even though GDPR/CCPA don't always classify it this way). Surface a separate consent.

## P3. Cross-border transfer — Art. 38, 39

**Difference**: Transfer of Chinese PI abroad requires one of:
- CAC security assessment (mandatory for Critical Information Infrastructure Operators and high-volume processors).
- A signed **Standard Contract** filed with CAC (template published by CAC in 2023).
- Personal information protection certification.
- Other conditions stipulated by law.

Plus, explicit separate consent from each user for the transfer.

**Engineering action**: For Chinese users, you must either localize data within China or implement one of the transfer mechanisms. The 2024 CAC clarifications relaxed thresholds for non-sensitive PI under 100,000 records, but still require disclosures. Engineering implication: be ready to route Chinese users to a region-local stack.

## P4. Data localization — Art. 40

**Difference**: Critical Information Infrastructure Operators (CIIOs) and processors handling above-threshold volumes must store PI of Chinese residents **within China**. Most consumer apps are not CIIOs but should plan for the possibility as they scale.

**Engineering action**: Architect for region routing. Avoid hardcoding a single region for user data storage. If targeting China at scale, set up a PRC-local environment.

## P5. Children — Art. 31

**Difference**: Under-**14** is the threshold. Specific, separate consent of a parent or guardian is required. Operators must adopt special rules for processing children's data.

**Engineering action**: Threshold differs from COPPA (US: 13) and GDPR (16, with member-state floor at 13). For Chinese users, use 14.

## P6. Right of explanation — Art. 24

**Difference**: For automated decisions with significant impact, individuals have a **right to refuse** decisions made solely by automated means **and** to require an explanation.

**Engineering action**: Same as GDPR Art. 22, but the refusal right is more explicit. Ensure a human-review path exists for any automated decision affecting Chinese users.

## P7. PIPO — Personal Information Protection Officer

**Difference**: Required when processing reaches CAC-defined volume thresholds (currently 1 million users of PI processed). Smaller scale: optional but recommended.

**Engineering action**: Confirm with ops as the user base grows; surface in privacy policy when applicable.

## P8. Mandatory in-China impact assessment — Art. 55, 56

**Difference**: A Personal Information Protection Impact Assessment (PIPIA, analogous to DPIA) is **mandatory** for processing sensitive PI, automated decisions, cross-border transfers, sharing with third parties, and other higher-risk activities.

**Engineering action**: For each such activity affecting Chinese users, ensure a PIPIA exists. Retain at least 3 years.

## P9. Breach notification — Art. 57

**Difference**: Notify both CAC and affected individuals **promptly** (no specific hours stipulated in the law, but CAC guidance points to immediate notification of the regulator for high-impact breaches; user notice may be omitted if effective measures have prevented harm).

**Engineering action**: Include CAC in the incident-response runbook. Be conservative — assume "promptly" means within 24–72 hours.

---

# DPDPA — India (Digital Personal Data Protection Act, 2023)

Enacted in August 2023. Rules under finalization (Draft DPDP Rules published January 2025 by MeitY; final rules expected during 2025). Enforced by the Data Protection Board of India. Some provisions are still subject to notification.

## D1. Lawful processing — Sec. 4, 7

**Difference**: DPDPA recognizes only two grounds: **consent** and "**legitimate uses**" (an enumerated list including state functions, employment, medical emergency, public-interest research). Legitimate-interest as found in GDPR is **not** a general lawful basis — narrower than GDPR.

**Engineering action**: For Indian users, default to explicit consent. Map every processing activity to either consent or one of the enumerated legitimate uses. Document the mapping.

## D2. Notice and consent quality — Sec. 5, 6

**Difference**: Notice must be **clear and itemized**, in English plus any of the 22 languages listed in the Eighth Schedule of the Constitution (user-selectable). Consent must be **free, specific, informed, unconditional, unambiguous, given by a clear affirmative action**.

**Engineering action**:
- Multi-language notice rendering for the 8th-Schedule languages where Indian users are targeted in volume.
- Notice must enumerate the personal data items, the specified purpose, and the manner of exercising rights.
- Avoid pre-ticked boxes; require an affirmative click.

## D3. Consent managers — Sec. 6(7), Draft Rules

**Difference**: India introduces **Consent Managers** — accountable third-party platforms registered with the Data Protection Board that mediate consent on behalf of users. The user can grant/withdraw consent across services through a single Consent Manager.

**Engineering action**: If operating at scale, plan API surface to accept consent records from a registered Consent Manager and to honor revocations received from one. The technical specifications are evolving — keep an eye on MeitY notifications.

## D4. Data Principal rights — Sec. 11, 12, 13

**Difference**: Rights are similar to GDPR (access, correction, erasure, grievance redressal, nomination of someone to exercise rights after death/incapacity). The **nomination** right is novel — users can name another person to exercise their rights.

**Engineering action**:
- Implement a `nominee` field at the user level, optional.
- The nominee may submit requests after a triggering event; build a separate validated path.

## D5. Children — Sec. 9

**Difference**: Under-**18** threshold. **Verifiable parental consent** required. Targeted advertising to children, tracking children, and behavioral monitoring of children are **prohibited**.

**Engineering action**:
- For Indian users, treat under-18 as children — much higher than GDPR/COPPA.
- Strong prohibition: do not run ad-tech, behavioral analytics, or profiling on under-18 Indian users at all.
- Verifiable parental consent — the rules contemplate using government-issued ID or DigiLocker-style verification; minimum: clearly more than a checkbox.

## D6. Significant Data Fiduciary — Sec. 10

**Difference**: The Board may designate a Data Fiduciary as "Significant" based on volume, sensitivity, risk to rights, electoral democracy impact, security of state, public order. SDFs have additional obligations: DPO appointment in India, periodic DPIA, periodic audit.

**Engineering action**: If your scale or product profile is high (large user base in India, sensitive sectors), plan for SDF obligations.

## D7. Cross-border transfer — Sec. 16

**Difference**: The Central Government may **notify** countries to which transfers are restricted or prohibited (it can publish either a whitelist or a blacklist; the operative approach is being finalized). Until notification, broad transfers are permitted, subject to other obligations.

**Engineering action**: Maintain the ability to route Indian user data to specific regions. Once the country list is published, you may need to disable or relocate flows quickly.

## D8. Breach notification — Sec. 8(6)

**Difference**: Data Fiduciary must notify the **Data Protection Board** and **each affected Data Principal** in case of a personal data breach. Specifics on timeline and content come from the rules — draft rules contemplate "without delay" and an interim report within 72 hours and detailed report within 30 days.

**Engineering action**: Add the Data Protection Board to the incident-response runbook. Plan for individual user notification (not just batch).

## D9. Penalties — Schedule (First)

Penalties are high: up to **₹250 crore** (approx US$30M) for failure to take reasonable security safeguards, up to **₹200 crore** for failure to notify breaches, etc. Engineering controls deserve real investment for Indian-targeted products.

---

# Market detection signals (reminder)

When deciding whether to load and apply sections of this file, use these signals from Phase 1:

- **LGPD**: locale `pt-BR`, currency `BRL`, `.br` domain, phone numbers with `+55`, mentions of "Brasil"/"Brazil" in app store metadata or marketing copy, Stripe/Mercado Pago configuration with Brazil, ANPD references.
- **PIPL**: locale `zh-CN`/`zh-Hans` (note: `zh-TW` is Taiwan, not PIPL), currency `CNY`, `.cn` domain, ICP filing references, WeChat/Alipay integration, Aliyun/Tencent Cloud configuration, phone numbers with `+86`.
- **DPDPA**: locale `hi`/`en-IN`/`bn`/`ta`/`te`/`mr`/`gu`/`kn`/`ml`/`pa`/`ur` and others from the 8th Schedule, currency `INR`, `.in` domain, phone numbers with `+91`, mentions of Aadhaar/UPI, DigiLocker integration, RBI references.

If signals are mixed or weak, ask the user explicitly which markets are in scope before applying these sections.
