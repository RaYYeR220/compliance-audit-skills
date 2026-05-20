# GDPR + UK GDPR Checklist

This file is loaded by `SKILL.md` during Phase 3. Each item is a concrete check with code patterns, severity, and a fix recommendation. UK GDPR mirrors EU GDPR in the articles cited here; report findings under both when an EU/UK scope is detected.

Apply the discovery map from Phase 2 against this checklist. Skip items marked "Not applicable" only when the underlying data simply isn't processed by the codebase.

---

## 1. Lawful basis for processing — Art. 6

**Check**: For every place where personal data is collected (signup form, lead capture, contact form, cookie set, analytics call), there must be a documented lawful basis: consent, contract performance, legal obligation, vital interests, public task, or legitimate interest.

**Look for**:
- A `consent` / `agreements` / `terms_accepted_at` field on the user model, or a separate consent record table.
- Comments or constants naming the basis (rare in practice — usually inferred).
- Cookie banner code paths that gate non-essential cookies.

**Red flags**:
- Analytics scripts loaded in `<head>` unconditionally with no consent gate (EU users).
- Marketing emails sent without an opt-in column or with a default `true`.
- Pre-ticked consent checkboxes in the signup form.
- Combining account creation consent with marketing consent in a single checkbox.

**Severity**: High when EU users are clearly in scope and no consent mechanism exists. Medium when a mechanism exists but is weak (pre-ticked, bundled).

**Fix recommendation**: Add a consent record table with `(user_id, purpose, version, granted_at, withdrawn_at, ip, source)`. Gate non-essential cookies and analytics behind explicit opt-in. Separate marketing consent from ToS acceptance.

---

## 2. Conditions for consent — Art. 7

**Check**: Where consent is the lawful basis, it must be freely given, specific, informed, and unambiguous; withdrawable as easily as given; and demonstrable.

**Look for**:
- A withdrawal/unsubscribe path in marketing emails (`/unsubscribe`, `?token=`).
- A "Manage cookies" link visible from any page, not only the initial banner.
- Consent records that include a version of the privacy policy text the user agreed to.

**Red flags**:
- Withdrawal requires emailing support (not "as easy as given").
- No record of *what version* of ToS/Privacy Policy was accepted.
- Single "I agree to everything" checkbox covering multiple purposes.
- Service unavailable unless marketing consent is granted (consent not freely given).

**Severity**: High when consent is bundled or withdrawal is harder than granting; Medium when version tracking is missing.

**Fix recommendation**: Add a public preferences page that lists each purpose and lets users toggle independently. Version your ToS and Privacy Policy and store the version string in each consent record. Provide a one-click unsubscribe in every marketing email.

---

## 3. Special category data — Art. 9

**Check**: Health, biometric, genetic, racial/ethnic origin, political opinions, religious beliefs, trade union membership, sex life, and sexual orientation require explicit consent (or another Art. 9(2) basis) and stronger safeguards.

**Look for**:
- Model fields named `health`, `medical`, `biometric`, `face_id`, `fingerprint`, `religion`, `ethnicity`, `political`, `union`, `sexual_orientation`, `gender_identity` (note: gender identity often treated as sensitive even when not strictly Art. 9).
- File upload paths for medical documents, ID photos, or selfies.
- ML models trained on sensitive attributes.

**Red flags**:
- Special category fields stored unencrypted.
- No explicit consent flow distinct from ordinary signup.
- Sensitive data sent to general-purpose analytics or LLMs.

**Severity**: Critical when special category data is unencrypted or exposed in logs. High when collected without explicit consent.

**Fix recommendation**: Apply column-level encryption. Restrict access via row-level security or service accounts. Add a separate explicit-consent UI step before collection. Audit-log every read.

---

## 4. Information to data subjects — Art. 12, 13, 14

**Check**: Users must be given specific information at collection (identity of controller, purposes, lawful basis, recipients, retention, rights, contact for complaints, DPO if applicable, international transfers).

**Look for**:
- A privacy policy file or route (`/privacy`, `privacy.md`, `privacy-policy.tsx`).
- Links to the policy from signup and account settings.
- A "data we collect from third parties" section if external sources are used.

**Red flags**:
- No privacy policy.
- Policy not linked from the signup form.
- Policy lists only generic info, not the specific data points the app collects.
- Last-updated date over 12 months old while the data model has changed since.

**Severity**: High when no privacy policy exists. Medium when policy exists but is generic or stale.

**Fix recommendation**: Maintain the privacy policy alongside the schema. Each new PII column triggers a policy update. Link from signup, footer, and account settings. Include controller identity, contact email, DPO contact if applicable, list of processors, retention periods, and the user's rights.

---

## 5. Right of access — Art. 15

**Check**: Users must be able to obtain a copy of all personal data held about them, plus metadata (purposes, recipients, retention, source).

**Look for**:
- An endpoint like `GET /api/me/data`, `GET /api/account/export`, `/gdpr/access`, or a "Download my data" button.
- A function that aggregates data from all tables that reference the user.

**Red flags**:
- No access endpoint.
- Endpoint returns only the row from the `users` table, missing related data (posts, orders, messages, audit logs).

**Severity**: High when no mechanism exists.

**Fix recommendation**: Implement an export endpoint that returns a structured JSON (or ZIP of JSON files) containing all rows referencing the user across the schema, plus a metadata block listing recipients and retention. Document the response with the schema in the privacy policy.

---

## 6. Right to rectification — Art. 16

**Check**: Users must be able to correct inaccurate data about themselves.

**Look for**:
- Account settings page allowing edits to name, email, address, phone, profile fields.
- API endpoints for updating user fields.

**Red flags**:
- Read-only profile fields that are PII (e.g. cannot change email or address).
- No way to fix data sourced from third parties.

**Severity**: Medium.

**Fix recommendation**: Make all collected PII fields editable from the account UI, or provide a contact path for fields that cannot be self-edited.

---

## 7. Right to erasure / "Right to be forgotten" — Art. 17

**Check**: Users must be able to request deletion of their personal data, unless an exemption applies (legal obligation, freedom of expression, public interest, legal claims).

**Look for**:
- A delete-account endpoint or UI (`/account/delete`, `/me/delete`, `deleteUser`, `closeAccount`).
- What the endpoint actually does: hard delete, soft delete with anonymization, or just flag.
- Cascade deletes / cleanup of related rows.
- Cleanup in third-party services: analytics, email lists, payment processor, CRM.

**Red flags**:
- No deletion endpoint at all.
- Soft delete only — row stays with PII intact and a `deleted_at` flag.
- Deletion removes the user row but leaves identifiable data in `events`, `audit_logs`, `messages`, `orders`, backups, third-party CRMs.
- Deletion is gated behind manual support request with multi-day SLA.

**Severity**: Critical when no deletion path exists or it does nothing meaningful. High when partial (account row deleted but related PII remains).

**Fix recommendation**: Implement an erasure flow that: (a) anonymizes user-referencing rows that must legally be kept (e.g. invoices) by replacing PII with non-identifying tokens; (b) hard-deletes everything else; (c) emits deletion events to downstream consumers (analytics, email provider, CRM, search index); (d) honors deletion within 30 days; (e) records the request and completion in a deletion log (encrypted, retained for proof).

---

## 8. Right to restriction — Art. 18

**Check**: Users can require that processing be paused while accuracy or legitimacy is disputed.

**Look for**: A user state flag like `processing_restricted`, `frozen`, `disputed`, and code paths that honor it.

**Severity**: Low to Medium. Often absent; flag as Low unless the product is in a regulated domain (finance, health).

**Fix recommendation**: Add a `processing_restricted_at` field, and a small middleware that returns 423 or skips downstream processing for restricted users.

---

## 9. Right to data portability — Art. 20

**Check**: Where processing is based on consent or contract and carried out by automated means, users can receive their data in a structured, commonly used, machine-readable format.

**Look for**: Whether the access endpoint (Art. 15) produces JSON/CSV (machine-readable) rather than PDF only.

**Red flags**: Export only as PDF, or only via screenshot/email.

**Severity**: Medium.

**Fix recommendation**: Produce JSON or CSV. PDF can be offered alongside but not as the only format.

---

## 10. Right to object — Art. 21

**Check**: Users can object to processing based on legitimate interest, public task, or direct marketing.

**Look for**:
- Marketing preferences toggle.
- Profiling for marketing toggle.
- A general "I object to processing for legitimate interest purposes" mechanism (rare in practice — often handled via support).

**Red flags**: Marketing opt-out exists but profiling/advertising opt-out does not.

**Severity**: Medium.

**Fix recommendation**: Add granular toggles per processing purpose. Honor objections immediately, not at next billing cycle.

---

## 11. Automated decision-making and profiling — Art. 22

**Check**: Users have a right not to be subject to decisions based solely on automated processing that produce legal or similarly significant effects (e.g. credit denial, hiring decision, insurance pricing).

**Look for**:
- ML models that make consequential decisions about users.
- Scoring engines: credit, risk, fraud, eligibility.
- LLM-based decision flows where the LLM output directly affects a user's account, eligibility, or pricing.

**Red flags**:
- Automated rejection (account, transaction, hiring) with no human-review path.
- No disclosure that an automated system is being used.
- LLM is the sole arbiter of a meaningful decision.

**Severity**: High when consequential decisions are fully automated with no human-review path.

**Fix recommendation**: Provide a clear path to request human review of the decision. Disclose the logic involved in the decision in plain language. Allow contesting the outcome. Document the safeguards in the privacy policy.

---

## 12. Data protection by design and by default — Art. 25

**Check**: The default configuration must minimize data collection and visibility.

**Look for**:
- Profile visibility defaults (public vs private).
- Tracking SDKs initialized with maximal data collection by default.
- Logging configured to capture full request bodies by default.

**Red flags**:
- New users default to public profiles.
- Tracking SDK auto-captures all DOM events and form inputs by default.
- Default log level captures full payloads including PII.

**Severity**: Medium.

**Fix recommendation**: Audit default settings. Profiles private by default unless the product is inherently public. Configure tracking SDKs to disable auto-capture of sensitive selectors. Lower default log verbosity in production.

---

## 13. Records of processing — Art. 30

**Check**: A record of processing activities must be maintained (controller, purposes, categories of data subjects and data, recipients, transfers, retention, security measures). Required for organizations of 250+ employees, or whenever processing is not occasional, or involves special category data.

**Look for**: A `data-processing-register.md`, `RoPA.md`, `processing-activities.yaml`, or equivalent in the repo or `docs/`.

**Severity**: Low (engineering checklist; usually maintained outside code). Note in report so operations can confirm it exists elsewhere.

**Fix recommendation**: Maintain a RoPA. If it lives outside the repo, link from the privacy policy.

---

## 14. Security of processing — Art. 32

**Check**: Appropriate technical and organizational measures: encryption, pseudonymization, confidentiality/integrity/availability, regular testing.

**Look for** (cross-reference with `security-checklist.md`):
- Password hashing algorithm.
- Encryption at rest for sensitive fields.
- TLS enforced for all client traffic.
- Secrets storage (env vars, secret manager) and absence of secrets in the repo.
- Backup encryption.

**Red flags**: covered in `security-checklist.md` in detail.

**Severity**: Critical for password/secret issues; High for missing encryption of sensitive data; Medium for missing security headers.

**Fix recommendation**: See `security-checklist.md`.

---

## 15. Personal data breach notification — Art. 33 and 34

**Check**: Controllers must notify the supervisory authority within 72 hours of becoming aware of a breach, and notify affected users without undue delay if there is high risk to their rights.

**Look for**:
- An incident response runbook (`SECURITY.md`, `incident-response.md`, `runbooks/breach.md`).
- Logging that would allow detecting and reconstructing a breach (audit logs of PII access).
- A mechanism to email all affected users in bulk.

**Red flags**:
- No incident response plan.
- No audit logs of PII reads, so the blast radius of a breach cannot be determined.
- No way to email affected users (e.g. transactional email provider not connected for bulk).

**Severity**: High when no plan exists. Medium when plan exists but logging is insufficient to support it.

**Fix recommendation**: Create a written incident response runbook with named roles, decision tree (72h vs immediate user notice), notification template, and supervisory authority contact. Ensure audit logs of PII access exist and are retained for at least 12 months.

---

## 16. DPIA — Art. 35

**Check**: A Data Protection Impact Assessment is required for high-risk processing: large-scale special category data, large-scale public monitoring, automated decisions with significant effects, profiling of children.

**Look for**: A `DPIA.md`, `dpia/`, or similar artifact.

**Severity**: Medium if the product clearly triggers DPIA requirements and no DPIA exists. Low otherwise.

**Fix recommendation**: When triggers apply, conduct a DPIA before launch. Template: necessity, proportionality, risks to subjects, mitigations.

---

## 17. Data Protection Officer — Art. 37

**Check**: A DPO is required for public authorities, organizations whose core activities consist of large-scale regular monitoring, or large-scale processing of special category data.

**Look for**: A `dpo@` email address or DPO contact in the privacy policy.

**Severity**: Low (engineering doesn't decide this).

**Fix recommendation**: Confirm with legal whether a DPO is required. If yes, publish the DPO contact in the privacy policy.

---

## 18. Processor obligations — Art. 28

**Check**: Every third party that processes user data on the controller's behalf must be under a Data Processing Agreement (DPA), bound to process only on instructions, with confidentiality and security obligations.

**Look for** (this is largely a paperwork check, but the code reveals which processors exist):
- All outbound SDK integrations identified in Phase 2: analytics, email, payments, customer support, error tracking, LLM providers, vector DBs, hosting, CDN, log aggregation.

**Severity**: High when a clearly PII-processing vendor is used and ops cannot confirm a DPA. Medium when partial.

**Fix recommendation**: Produce a vendor list from the codebase. Cross-check with operations: signed DPA on file? If not, halt or switch vendors. Include sub-processors disclosure in the privacy policy.

---

## 19. International transfers — Art. 44–49

**Check**: Transfers of EU/UK personal data to third countries (anything outside EEA + UK + countries with adequacy decisions) must use appropriate safeguards: Standard Contractual Clauses, Binding Corporate Rules, or an adequacy decision. Schrems II requires a Transfer Impact Assessment for US transfers.

**Look for**:
- Hosting region settings (`AWS_REGION`, `GCP_PROJECT`, Vercel regions, Supabase region).
- Database location.
- Analytics endpoints (often US-hosted).
- LLM API endpoints (OpenAI, Anthropic — US-hosted by default unless EU residency option used).

**Red flags**:
- EU users' data routed through US-hosted services with no SCCs disclosed.
- Logs shipped to a US-region datadog/sentry account with no DPA.

**Severity**: High when transfers are unmistakable and no safeguards are visible in policy. Medium when ambiguous.

**Fix recommendation**: List every cross-border data flow. For each US destination, ensure SCCs are signed and a TIA is completed. Consider EU residency options where the vendor provides them (Anthropic EU, OpenAI EU data residency, Sentry EU). Disclose transfers in the privacy policy.

---

## 20. Children — Art. 8

**Check**: For information society services offered directly to children, consent of a parent/guardian is required for children under 16 (EU baseline; member states may lower to 13). UK GDPR sets the threshold at 13.

**Look for**:
- Age gates at signup, date of birth field.
- Product positioning indicating teen/child audience.
- Parental consent workflows.

**Red flags**:
- App clearly attractive to minors with no age gate.
- Age field collected but never used to branch consent flow.

**Severity**: Critical if the app is directed at children and no parental consent flow exists; otherwise Low.

**Fix recommendation**: Add an age gate at signup. For users under the relevant threshold, require verifiable parental consent and apply stricter defaults. Reference COPPA for US (separate law — see CCPA checklist note).

---

## 21. Cookie / ePrivacy gate

**Check**: ePrivacy Directive (separate from GDPR but enforced alongside in EU) requires prior, informed, freely-given consent for non-essential cookies and similar tracking technologies, before they are set.

**Look for**:
- Cookie banner implementation: when does it appear, what does it block?
- Whether tracking scripts (`gtag`, `fbq`, `mixpanel`, `hotjar`, `clarity`) are inserted before consent.
- Whether the banner offers "Reject all" as easily as "Accept all".

**Red flags**:
- Tracking scripts load on page load with no consent gate.
- Banner offers only "Accept all" prominently; "Reject" hidden in a sub-menu.
- Banner sets cookies before user interaction.

**Severity**: High in EU/UK scope when non-essential cookies fire pre-consent.

**Fix recommendation**: Block all non-essential trackers behind a consent state. Banner should offer "Accept all" and "Reject all" with equal prominence and a "Customize" option. Record consent decisions per category (necessary, analytics, marketing) with a version.

---

## 22. Data minimization in API responses

**Check**: API responses should return only the fields the caller needs.

**Look for**:
- Endpoints returning a full `User` object including email, phone, internal IDs, password hash columns, when the caller (public profile view) only needs a username.
- Admin fields leaking into user-facing responses.

**Red flags**:
- `SELECT *` from users endpoints.
- Serializers that include internal columns.
- Public endpoints returning private fields of other users.

**Severity**: High when other users' PII is exposed. Medium when over-exposed to the same user.

**Fix recommendation**: Define explicit response schemas/DTOs. Default to allow-list, not deny-list. Add tests that assert sensitive fields are not present in public responses.

---

## 23. Retention periods

**Check**: Personal data must not be kept longer than necessary for the stated purpose.

**Look for**:
- Cron jobs, scheduled tasks, or background workers that delete old records.
- Retention configuration in storage (S3 lifecycle, log retention settings).
- A documented retention schedule.

**Red flags**:
- All data retained indefinitely.
- Inactive accounts never archived or deleted.
- Logs kept for years with PII intact.

**Severity**: Medium.

**Fix recommendation**: Define a retention schedule per data category (e.g. inactive accounts purged after 24 months of inactivity; web access logs 90 days; transactional logs 7 years for tax). Implement as scheduled jobs. Document in the privacy policy.
