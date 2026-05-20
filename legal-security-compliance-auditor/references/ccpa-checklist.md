# CCPA / CPRA Checklist (California)

This file is loaded by `SKILL.md` during Phase 3 when the US scope is active. CCPA was amended by CPRA effective January 1, 2023; this checklist reflects the amended law. Cite both as "CCPA/CPRA" or use the specific section.

CCPA applies to for-profit businesses that do business in California and meet at least one of: (a) gross revenue over $25M, (b) buy/sell/share PI of 100,000+ consumers or households annually, (c) derive ≥50% of revenue from selling/sharing PI. If the project is a startup that does not yet meet thresholds, flag the findings at Low/Medium with a note: *"applies once the business meets a CCPA threshold."* Still report — most products grow into it.

Several other US states have CCPA-like laws (Virginia VCDPA, Colorado CPA, Connecticut CTDPA, Utah UCPA, Texas TDPSA, Oregon OCPA, etc.). The same engineering controls satisfy most of them; mention this in the report rather than repeating checks per state.

---

## 1. Categories of PI handled

**Check**: Identify and classify the personal information categories handled. CCPA defines categories broadly (identifiers, commercial info, biometric, internet activity, geolocation, sensory data, professional/employment, education, inferences, sensitive PI).

**Look for**: Same discovery output as GDPR Phase 2 — map PII fields to CCPA categories. Note any **sensitive PI** specifically: SSN, driver's license, financial account, precise geolocation, racial/ethnic origin, religious beliefs, union membership, communications contents, genetic data, biometric data, health, sex life, sexual orientation.

**Severity**: Low (classification, not a finding). Used as input to later checks.

**Output to report**: A short table mapping detected fields to CCPA categories. This belongs in the report's "Stack detected" or a dedicated subsection.

---

## 2. Notice at collection — CCPA §1798.100(b)

**Check**: At or before the point of collection, the business must inform the consumer of: (a) categories of PI collected, (b) purposes for which each category is used, (c) whether each category is sold or shared, (d) retention period for each category.

**Look for**:
- A short notice near each PII collection form (signup, contact, checkout, lead capture).
- Privacy policy linkage from forms.
- A `data-categories` or similar configuration mapping each field to a purpose.

**Red flags**:
- Forms collect PII with only "By clicking Submit you agree to our Privacy Policy" — no enumeration of purposes at point of collection.
- New PII fields added with no update to the notice or policy.

**Severity**: Medium when missing; High when forms collect sensitive PI with no notice.

**Fix recommendation**: Add a brief just-in-time notice near each form field group, linked to the full privacy policy. Maintain the purpose-per-field mapping in a config that drives both the notice and the policy.

---

## 3. Privacy policy disclosures — CCPA §1798.130

**Check**: The privacy policy must include: categories of PI collected in the past 12 months, sources, business/commercial purposes, categories of third parties to whom PI is disclosed/sold/shared, and the consumer rights (with how to exercise them).

**Look for**: Privacy policy file or route. Compare against the actual data model — does it list everything actually collected?

**Red flags**:
- Privacy policy uses generic template language and does not match the actual data collected.
- No mention of how to submit a consumer request.
- "Last updated" over 12 months old or older than the most recent schema change to a PII model.

**Severity**: High when the policy is materially out of date or omits required disclosures.

**Fix recommendation**: Keep the privacy policy in the repo alongside schema. Treat changes to PII models as triggering a policy update. Include the categories from §1798.130, plus methods for consumer requests (web form, email, toll-free number if revenue requires).

---

## 4. Right to know — CCPA §1798.110, §1798.115

**Check**: Consumers can request: (a) the categories and specific pieces of PI collected, (b) sources, (c) business/commercial purposes, (d) categories of third parties with whom shared. Must be honored within 45 days (extendable once by 45 days).

**Look for**:
- An access endpoint or form (`/api/me/data`, `/privacy-request`, a CSAT request form for non-account-holders).
- A method to handle requests from non-account-holders (CCPA covers consumers, not just authenticated users).
- Identity verification for the request (CCPA requires verification, especially for sensitive PI).

**Red flags**:
- No request mechanism.
- Request mechanism exists only for logged-in users — visitors who never created an account cannot request.

**Severity**: High when no mechanism exists.

**Fix recommendation**: Provide a public privacy request portal that accepts requests from non-account-holders. Implement identity verification proportional to sensitivity. Document SLAs in the privacy policy.

---

## 5. Right to delete — CCPA §1798.105

**Check**: Consumers can request deletion of PI. Exceptions apply (transaction completion, legal obligation, security incidents, debugging, free speech, internal uses reasonably aligned with consumer expectations, comply with legal obligations).

**Look for**: Same as GDPR Art. 17 — see `gdpr-checklist.md` item 7. Cite both laws in the finding.

**Red flags**: Same.

**Severity**: Critical when no deletion path exists.

**Fix recommendation**: Same as GDPR Art. 17 fix. Add to the privacy request portal. Service providers and contractors must propagate deletion.

---

## 6. Right to correct — CCPA §1798.106 (CPRA addition)

**Check**: Consumers can request correction of inaccurate PI. Effective from CPRA.

**Look for**: Same as GDPR Art. 16 — editable fields, plus a request path for non-self-editable fields.

**Severity**: Medium.

**Fix recommendation**: Editable account UI for PII. Privacy request portal accepts correction requests for non-self-editable data.

---

## 7. Right to opt-out of sale or sharing — CCPA §1798.120, §1798.135

**Check**: Consumers can opt out of "sale" or "sharing" of PI. "Sharing" was added by CPRA and explicitly covers cross-context behavioral advertising. Most ad-tech and many analytics integrations qualify.

**Look for**:
- Whether the site embeds ad-tech (Meta Pixel, Google Ads, TikTok Pixel, LinkedIn Insight, Twitter pixel, TradeDesk, Criteo), Google Analytics with advertising features enabled, or "Lookalike" integrations.
- Whether a "Do Not Sell or Share My Personal Information" link is present in the footer and accessible from any page.
- Whether the opt-out is honored: does the chosen consent state actually block the tracker?

**Red flags**:
- Ad-tech embedded, no "Do Not Sell or Share" link.
- Link exists but opting out does nothing — pixel still fires.
- Link buried in the privacy policy instead of in the global footer.

**Severity**: High when ad-tech is present and no opt-out is provided.

**Fix recommendation**: Add a "Do Not Sell or Share My Personal Information" link in the global footer. The link should set a persistent preference (cookie + server-side record). All sharing trackers must be gated by this preference. Honor the Global Privacy Control header (see item 10).

---

## 8. Right to limit use of sensitive PI — CCPA §1798.121 (CPRA addition)

**Check**: If sensitive PI is used or disclosed beyond the purposes necessary to provide the service, consumers can require limitation to those necessary purposes.

**Look for**:
- Whether sensitive PI fields (SSN, precise geolocation, financial account, health, biometrics) are used for purposes beyond service delivery (e.g. profiling, marketing).
- Whether a "Limit the Use of My Sensitive Personal Information" link is present when applicable.

**Red flags**:
- Sensitive PI used for marketing or profiling without disclosure or limit.
- No limit-use link when sensitive PI is used beyond strictly necessary purposes.

**Severity**: High when applicable and absent.

**Fix recommendation**: Restrict sensitive PI to service-delivery purposes by default. If broader use is required, add the limit-use link and honor it.

---

## 9. Right to non-discrimination — CCPA §1798.125

**Check**: A business cannot discriminate against a consumer for exercising rights (deny goods/services, charge different prices, provide different quality). Financial incentives are permitted if reasonably related to value of the data.

**Look for**:
- Feature gating based on consent (e.g. "must accept marketing to use the free tier").
- Different pricing based on opt-out status.

**Red flags**: Free tier requires marketing opt-in. Service degraded for users who opt out of sharing.

**Severity**: High when present.

**Fix recommendation**: Remove gating based on rights exercise. If a loyalty program offers benefits in exchange for data, structure as a documented "financial incentive" with disclosure.

---

## 10. Universal opt-out / Global Privacy Control — CCPA Regulations §7025

**Check**: Businesses must honor the Global Privacy Control (`Sec-GPC: 1`) signal as an opt-out of sale/sharing.

**Look for**:
- Middleware or analytics setup that reads the `Sec-GPC` header.
- A check for `navigator.globalPrivacyControl` in client code before initializing trackers.

**Red flags**: GPC ignored entirely while ad-tech is present.

**Severity**: High when GPC is ignored and sharing trackers exist.

**Fix recommendation**: At the earliest possible point, read `Sec-GPC` (server) or `navigator.globalPrivacyControl` (client). When `true`, set the opt-out state and refuse to initialize sharing trackers. Document this behavior in the privacy policy.

---

## 11. Minor opt-in for sale/sharing — CCPA §1798.120(c)

**Check**: For consumers under 16, opt-in (rather than opt-out) is required for sale or sharing. For under-13, parental consent is required.

**Look for**:
- Age-related fields and how they gate consent.
- Whether age is collected at all (for products that should know).

**Red flags**: No age gate while ad-tech is active and the product appeals to minors.

**Severity**: Critical when minors are clearly in scope and no opt-in flow exists.

**Fix recommendation**: Add age gating. Under-13 users require verifiable parental consent (also a COPPA requirement — federal). 13–16 users require explicit opt-in for sale/share. Default to opt-out for minors.

---

## 12. Service providers and contractors — CCPA §1798.140(ag), §1798.140(j); §1798.105(c)

**Check**: PI shared with vendors must be governed by a written contract restricting use to the specified purposes and prohibiting selling/sharing. Deletion requests must be propagated.

**Look for**:
- The vendor list derived from Phase 2 discovery.
- Documentation or known DPA/SPA status for each vendor.

**Red flags**: PII-handling vendors with no documented contract terms (engineering can't verify; flag for ops).

**Severity**: Medium (process-level; engineering surfaces it for ops to confirm).

**Fix recommendation**: Cross-check vendor list with operations. Ensure each PII-handling vendor has a signed contract with CCPA service-provider language. Implement deletion-request propagation API calls where supported (Segment Tracking Plan + Source Functions, Hubspot, Intercom, Sentry, Datadog all have deletion APIs).

---

## 13. Request response timeline — CCPA §1798.130

**Check**: Verifiable consumer requests must be confirmed within 10 business days and responded to within 45 calendar days (extendable once by 45 days).

**Look for**:
- An automated acknowledgment after a request is submitted.
- An internal ticket / queue for tracking SLA.

**Severity**: Low-Medium. Engineering check is whether the system supports the SLA, not whether ops meets it.

**Fix recommendation**: Automate the acknowledgment email immediately on request. Track requests in a queue with SLA timers.

---

## 14. Identity verification — CCPA Regulations §7060

**Check**: Verifying the identity of a requestor before fulfilling. Authentication of account-holders is generally sufficient for non-sensitive data; sensitive requests need stronger verification.

**Look for**:
- Whether logged-in users get rate-limited or require re-authentication for sensitive actions (data export, deletion).
- Whether non-account holders provide additional information.

**Red flags**: Deletion or export available immediately on session cookie, no re-auth.

**Severity**: Medium.

**Fix recommendation**: Step-up authentication (re-enter password or 2FA) for export and deletion. For non-account-holders, require matching at least two pieces of information held on file.

---

## 15. Record retention — CCPA Regulations §7101

**Check**: Records of consumer requests and responses must be maintained for 24 months.

**Look for**:
- A `privacy_requests` or `dsar` table that captures the request, response, date, verification status.

**Severity**: Low.

**Fix recommendation**: Add a request log table. Retain for 24 months minimum. Restrict access.

---

## 16. Cookies / similar technologies as PI

**Check**: Persistent identifiers (cookies, device IDs) are explicitly PI under CCPA. Most cookie-banner work done for GDPR partially covers this, but CCPA's "sale/share" frame is different — many ad-tech cookies are "shares" even where users have not opted out.

**Look for**: Same as GDPR cookie check (item 21 in `gdpr-checklist.md`).

**Severity**: cross-reference, do not double-count. Note CCPA implication in the GDPR finding.

---

## 17. Data broker registration — CA Civ. Code §1798.99.80–.88

**Check**: If the business sells PI of consumers with whom it does not have a direct relationship, it must register annually as a data broker with the CPPA.

**Look for**: B2B lead-data products, scraping pipelines, identity-graph products.

**Severity**: High when applicable. Flag for legal/ops.

**Fix recommendation**: Confirm data broker status with legal. Register if applicable.
