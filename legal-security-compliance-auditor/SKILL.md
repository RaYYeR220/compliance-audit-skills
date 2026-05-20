---
name: legal-security-compliance-auditor
description: Audits a codebase for data privacy law compliance (GDPR, UK GDPR, CCPA/CPRA, optionally LGPD/PIPL/DPDPA) and data security. Adapts scope automatically by detecting the project's target markets from i18n files, currencies, language, framework signals, and domain hints. Inspects database models, API endpoints, forms, logs, third-party integrations, LLM/AI usage, cookies, and authentication flows. Checks user rights endpoints (deletion, export, access), encryption at rest and in transit, password hashing, consent mechanisms, breach notification readiness, AI disclosure, and PII leakage. Produces a single severity-graded COMPLIANCE_AUDIT.md report with Critical/High/Medium/Low findings, file:line citations, the specific law article cited, and concrete remediation steps. Use this skill when the user asks to audit privacy, check GDPR or CCPA compliance, review for legal launch, prepare for a data protection review, find PII or personal data issues, audit data security, check AI/LLM disclosure, review the app before launching in the EU/UK/US, prepare for production with user data, do a compliance check, run a privacy audit, or look for legal red flags.
---

# Legal, Security & Compliance Auditor

You are running a privacy and data security audit of the current codebase. Your goal is to find concrete, citable issues that would cause legal exposure or a failed compliance review, and produce a clean, prioritized report.

## Important boundary

This skill produces a **compliance audit**, not legal advice. Every report you produce must end with a disclaimer: *"This audit is an engineering checklist, not legal counsel. Consult a qualified data protection lawyer for binding decisions on your specific jurisdiction and product."*

Do not invent legal opinions. When a finding depends on legal interpretation, flag the ambiguity rather than asserting a verdict.

## Workflow

Execute these phases in order. Do not skip ahead.

### Phase 1 — Scope detection

Before auditing, determine the project's target markets. This decides which extended laws to apply.

1. Read manifest files if they exist: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, `*.csproj`, `pubspec.yaml`. Note the primary language and frameworks.
2. Search the repository for these market signals:
   - **EU / UK signals**: `.de`, `.fr`, `.eu`, `.co.uk` domains in env/config; currencies `EUR`, `GBP`; i18n locales `de`, `fr`, `es`, `it`, `pl`, `nl`, `en-GB`; mentions of `GDPR`, `cookie consent`, `EEA`, `DSGVO`, `ICO`.
   - **US signals**: `.com` (default, not strong), currency `USD`, locale `en-US`, mentions of `CCPA`, `CPRA`, `California`, `"Do Not Sell"`.
   - **Brazil (LGPD)**: locale `pt-BR`, currency `BRL`, `.br` domain, mentions of `LGPD`, `ANPD`.
   - **China (PIPL)**: locale `zh-CN`, `zh-Hans`, currency `CNY`, `.cn` domain, mentions of `PIPL`, `CAC`, ICP filing.
   - **India (DPDPA)**: locale `hi`, `en-IN`, currency `INR`, `.in` domain, mentions of `DPDPA`, `consent manager`.
3. Identify the application kind: backend API, full-stack web, server-side rendered (SSR) framework, static site with analytics, mobile backend.
4. Detect data layer technology: ORMs (Prisma, Drizzle, TypeORM, Sequelize, Mongoose, SQLAlchemy, Django ORM, Tortoise, Active Record, Ent, GORM), raw SQL, NoSQL clients.
5. Detect authentication: Auth0, Clerk, Supabase Auth, NextAuth, Lucia, Passport, Devise, custom JWT, custom sessions.
6. Detect LLM/AI usage: SDK imports for `openai`, `@anthropic-ai/sdk`, `@google/generative-ai`, `cohere`, `mistralai`, `langchain`, `llamaindex`, `vercel/ai`, ollama clients, vector stores (`pinecone`, `weaviate`, `chroma`, `pgvector`).
7. Detect analytics and tracking: `segment`, `mixpanel`, `amplitude`, `posthog`, `google-analytics`, `gtag`, `facebook-pixel`, `hotjar`, `fullstory`, `clarity`, `sentry`, `datadog`.

**Required scope baseline** (always applied): GDPR + UK GDPR + CCPA/CPRA + security + LLM (if AI detected).

**Conditional scope** (applied only when detected or user-confirmed): LGPD, PIPL, DPDPA.

If signals are ambiguous, ask the user one short question: *"I see signals for [list]. Which markets do you plan to ship to? (EU, UK, US California, Brazil, China, India, other)"* Then proceed with the confirmed scope.

State the determined scope to the user before starting Phase 2.

### Phase 2 — Discovery

Map every location where personal data lives or moves through the code. Personal data ("PII") includes any of: name, email, phone, postal address, IP address, device IDs, government IDs, payment data, precise geolocation, biometrics, health, ethnicity, sexual orientation, political opinions, religion, photos of people, behavioral profiles, cookie identifiers, user-generated content tied to identity.

Search the codebase for:

1. **Database schemas**:
   - ORM model definitions (`*.prisma`, `models.py`, `*.entity.ts`, `schema.ts`, `*.model.js`, `*Model.swift`, `*.rb` under `app/models`, `models/*.go`).
   - Migration files in `migrations/`, `db/migrate/`, `prisma/migrations/`, `alembic/`.
   - Note every column that holds PII. Flag fields without encryption or hashing.
2. **API endpoints accepting or returning PII**:
   - Route files (`routes/`, `api/`, `controllers/`, `app/api/`, Next.js `app/**/route.ts`, FastAPI `@app.post`, Django `views.py`/`urls.py`, Express handlers, NestJS controllers, Rails `*_controller.rb`).
   - Look for request bodies, query params, and responses containing PII fields.
3. **Forms collecting PII**:
   - HTML forms, React forms (`<input>`, `react-hook-form`, `formik`), mobile forms.
   - Check whether each field has a stated purpose and whether collection is minimal.
4. **Logging code**:
   - Calls like `console.log`, `logger.info`, `print`, `puts`, `log.Println`, `Rails.logger`, structured loggers (`pino`, `winston`, `bunyan`, `loguru`, `zap`, `zerolog`).
   - Flag any log call whose interpolated arguments include user objects, request bodies, headers (`Authorization`, `Cookie`), tokens, emails, or IDs.
5. **Third-party data sharing**:
   - Outbound HTTP calls (`fetch`, `axios`, `requests`, `httpx`, `net/http`) to external services.
   - Analytics SDK initializations and `identify(user)` / `track(event, properties)` calls.
   - Payment processors (Stripe, PayPal, Adyen, Square).
   - Email/SMS providers (SendGrid, Postmark, Mailgun, Twilio, AWS SES).
   - Background job payloads and webhook handlers.
6. **Authentication & session storage**:
   - Password hashing function used. Acceptable: bcrypt, argon2, scrypt, pbkdf2 with high iterations. Unacceptable: md5, sha1, sha256 alone, plaintext.
   - Session/JWT storage location and contents.
   - Token expiration and refresh strategy.
7. **Cookies and tracking pixels**:
   - `Set-Cookie` headers, `document.cookie`, cookie libraries.
   - Tag managers (`gtm.js`, `Google Tag Manager`).
   - Tracking scripts loaded before consent.
8. **File uploads**:
   - Upload endpoints, storage drivers (S3, GCS, Azure Blob, Supabase Storage, Cloudinary).
   - Whether uploaded files containing PII (IDs, selfies) are encrypted at rest and access-controlled.
9. **LLM/AI surfaces** (if detected in Phase 1):
   - Where prompts are constructed and what user data is included.
   - Whether responses are stored, and for how long.
   - Whether the user-facing UI discloses that AI is generating content.
10. **User rights endpoints**:
    - Search for routes containing `delete-account`, `deleteUser`, `gdpr`, `dsr`, `data-export`, `download-my-data`, `account/delete`, `me/delete`.
    - Record whether deletion and export endpoints exist, and what they actually do (soft delete vs hard delete vs cascade).
11. **Consent records**:
    - Tables or files tracking consent (`consents`, `user_consents`, `cookie_preferences`, `agreements`).
    - Whether consent is timestamped, versioned (against current ToS/Privacy version), and revocable.
12. **Public legal documents**:
    - Privacy Policy and Terms of Service files or routes (`/privacy`, `/terms`, `privacy-policy.md`, `legal/`).
    - Whether they are linked from signup and account pages.

Produce an internal map of findings before moving to Phase 3. Do not output it yet.

### Phase 3 — Audit

For each applicable scope determined in Phase 1, load and apply the relevant checklist:

- Always load `references/gdpr-checklist.md` (covers GDPR + UK GDPR; they share most articles).
- Always load `references/ccpa-checklist.md`.
- Always load `references/security-checklist.md`.
- Load `references/llm-checklist.md` if LLM/AI integrations were detected.
- Load `references/extended-laws.md` if LGPD, PIPL, or DPDPA is in scope.

For every check item in the loaded references, run it against the discovery map from Phase 2. Record:
- Whether it passes, fails, or is not applicable.
- For failures: the exact `file:line` location, the article or section cited, what is wrong, severity, and the recommended fix.

**Severity definitions** (use consistently):

- **Critical** — High likelihood of legal action, regulatory fine, or data breach. Examples: plaintext passwords, no deletion endpoint at all, sending PII to third parties with no agreement, secrets committed to git.
- **High** — Clear non-compliance with a specific law that would fail a regulator audit. Examples: missing data export, no breach notification process, no consent records, tracking before consent in EU.
- **Medium** — Practice that is non-compliant in spirit or in a strict reading but unlikely to trigger immediate enforcement. Examples: privacy policy outdated, retention period not documented, weak cookie banner.
- **Low** — Best-practice gap or hygiene issue. Examples: missing security headers, no documented DPO contact, PII column names not commented as PII.

### Phase 4 — Report generation

Write the audit to `COMPLIANCE_AUDIT.md` at the repository root (overwrite if it exists; mention this to the user before writing).

Use exactly this structure:

```markdown
# Compliance Audit

**Generated**: <ISO date>
**Scope**: <list of laws applied, e.g. "GDPR, UK GDPR, CCPA/CPRA, Security, LLM Disclosure">
**Stack detected**: <e.g. "Next.js 14, Prisma + PostgreSQL, Clerk auth, OpenAI SDK">
**Target markets**: <e.g. "EU, UK, US (California)">

## Summary

| Severity | Count |
|----------|-------|
| Critical | N     |
| High     | N     |
| Medium   | N     |
| Low      | N     |
| **Total**| N     |

## Critical

### C-1. <Short title of finding>
- **Law**: <e.g. "GDPR Art. 32 — Security of processing">
- **Location**: `path/to/file.ts:42`, `path/to/other.ts:108`
- **What's wrong**: <one or two sentences>
- **Why it matters**: <one sentence>
- **Recommended fix**: <concrete code-level or process-level action>

### C-2. ...

## High

### H-1. ...

## Medium

### M-1. ...

## Low

### L-1. ...

## Passed checks

A short bullet list of checks that passed, grouped by area (Database, Auth, User rights, Consent, AI, Logging, Third parties). Keep it terse — one line per check.

## Not applicable

Short bullet list of checks that didn't apply and why (e.g. "Children's data — no minors targeted per ToS").

## Next steps

A 3-5 step priority order for remediation, starting with all Critical items.

---

*This audit is an engineering checklist, not legal counsel. Consult a qualified data protection lawyer for binding decisions on your specific jurisdiction and product.*
```

Rules for the report:
- Cite real `file:line` for every finding. If multiple locations, list up to three with "and N more" if needed.
- Quote the relevant law article precisely (e.g. "GDPR Art. 17(1)(a)" not just "GDPR").
- The recommended fix must be specific. Bad: *"add encryption"*. Good: *"encrypt the `users.phone` column using application-level encryption (e.g. `@daiyam/aes-gcm` or column-level encryption with Postgres pgcrypto). Decrypt only at the application layer; never log decrypted values."*
- Do not fabricate findings. If you cannot find evidence of a problem, do not invent one.
- If a check requires information you cannot derive from code (e.g. whether a DPA is signed with a processor), classify it as Medium with an action: *"Verify with operations: signed DPA on file with [vendor]."*

### Phase 5 — Summary to user

After writing the file, give the user a 5-7 line summary:
- Where the report was written.
- Total count by severity.
- The single highest-priority Critical issue (if any).
- Recommended immediate next action.
- Reminder that this is an engineering checklist, not legal advice.

Do not paste the full report into chat. Direct the user to the file.

## Output format examples

A good Critical finding:

```markdown
### C-1. Passwords stored as unsalted SHA-256
- **Law**: GDPR Art. 32(1)(a) — appropriate technical measures; CCPA §1798.150 (private right of action for breach due to lack of reasonable security).
- **Location**: `src/auth/register.ts:34`, `src/auth/login.ts:21`
- **What's wrong**: `crypto.createHash('sha256').update(password).digest('hex')` is used to store and compare passwords. SHA-256 is not designed for password hashing — it is fast and unsalted, enabling efficient rainbow-table and GPU attacks.
- **Why it matters**: A breach would expose all user passwords in days. This fails GDPR Art. 32 "state of the art" requirement and exposes you to CCPA's private right of action.
- **Recommended fix**: Replace with `argon2id` (preferred) or `bcrypt` with cost ≥ 12. Use `node-argon2` or `bcrypt` package. On next login, re-hash and store; on registration, hash immediately. Migrate existing hashes by re-hashing at next successful login.
```

A weak finding to avoid:

```markdown
### Bad example — do not do this
- **Law**: GDPR
- **Location**: server.js
- **What's wrong**: Code might not be GDPR compliant.
- **Recommended fix**: Make it GDPR compliant.
```

## Edge cases and rules of thumb

- **Greenfield / empty project**: If the codebase has no user data handling at all, report that and exit gracefully. Do not invent findings.
- **Monorepo**: Run discovery per package. Report findings with the package name in the location (e.g. `apps/web/src/...`).
- **Generated code**: Skip `node_modules`, `.next`, `dist`, `build`, `vendor`, `target`, `__pycache__`, `.venv`. If lockfiles contain dependency names relevant to scope (e.g. an analytics SDK), still report on the dependency.
- **Test files**: Findings in test fixtures (e.g. mock emails in `*.test.ts`) are not findings. Findings in test code that exercises production code paths are.
- **Existing audit file**: If `COMPLIANCE_AUDIT.md` already exists, ask the user whether to overwrite or append a dated section.
- **Very large codebases**: If discovery finds 50+ instances of the same pattern (e.g. logging), collapse to a single finding with "and X more" plus the top three locations.
- **Conflicting laws**: If GDPR and CCPA require different behavior, list both requirements and recommend the stricter one (usually GDPR).
- **User pushes back on a finding**: Lower confidence is acceptable. Move it to a section "Disputed / needs review" with the user's reasoning recorded.

## What this skill does NOT do

- Does not write or modify application code. It only writes the audit report.
- Does not generate Privacy Policy or Terms of Service text. Recommend a lawyer or a vetted template for that.
- Does not perform dynamic testing, penetration testing, or runtime analysis. Static review only.
- Does not assess physical security, employee training, or business processes outside the codebase.
- Does not certify the application as compliant. Only a qualified DPO and lawyer can do that.
