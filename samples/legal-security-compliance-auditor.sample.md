# Compliance Audit

**Generated**: 2026-05-19
**Scope**: GDPR, UK GDPR, CCPA/CPRA, Security, LLM Disclosure
**Stack detected**: Next.js 14 (App Router), TypeScript, Prisma + PostgreSQL, Clerk auth, OpenAI SDK (`gpt-4o`), Stripe, PostHog, Sentry. Hosting: Vercel (US-East). Database: Supabase (US-East).
**Target markets**: EU, UK, US (California). LGPD/PIPL/DPDPA not in scope.

## Summary

| Severity | Count |
|----------|-------|
| Critical | 3     |
| High     | 6     |
| Medium   | 7     |
| Low      | 4     |
| **Total**| 20    |

## Critical

### C-1. Full request bodies logged in production, including passwords on signup
- **Law**: GDPR Art. 32(1)(a); CCPA §1798.150
- **Location**: `src/middleware.ts:18`, `src/lib/logger.ts:42`
- **What's wrong**: The request middleware emits `logger.info('request', { method, url, body })` for every POST. On `/api/auth/sign-up`, the body contains `password` in plaintext. Logs ship to Sentry and to Vercel's log stream with 30-day retention.
- **Why it matters**: A single log breach exposes every signup password from the retention window. This fails GDPR Art. 32 "state of the art" and exposes you to CCPA's private right of action for breach due to lack of reasonable security.
- **Recommended fix**: Maintain a deny-list of fields stripped before logging (`password`, `token`, `secret`, `authorization`, `cookie`, `ssn`, payment fields). Apply at the logger level, not per-route. Add a `Sentry.beforeSend` hook that scrubs the same fields from event payloads. Rotate any passwords that were captured during the current retention window.

### C-2. No account deletion endpoint
- **Law**: GDPR Art. 17(1); CCPA §1798.105
- **Location**: `src/app/api/account/` (no `delete` route), `src/app/(account)/settings/page.tsx:1-180` (no delete UI)
- **What's wrong**: There is no path — UI or API — for a user to delete their account. The schema also has no `deleted_at` or anonymization helpers.
- **Why it matters**: GDPR's right to erasure is unsatisfied. A regulator complaint or DSAR would fail immediately. CCPA imposes the same obligation for California users.
- **Recommended fix**: Implement `DELETE /api/account` that: (a) requires fresh re-auth, (b) anonymizes records that must legally be retained (e.g. `Invoice` rows — replace `userId` with a non-identifying token), (c) hard-deletes everything else via Prisma `onDelete: Cascade` relations, (d) emits a deletion event to Clerk, PostHog, Stripe customer, and Sentry user, (e) records the request in a `DeletionLog` table for proof.

### C-3. OpenAI API key committed to repository history
- **Law**: GDPR Art. 32(1)(b); CCPA §1798.150
- **Location**: `apps/web/.env.example:5` (placeholder fine), but `git log --all -S 'sk-proj-'` reveals a live key in commit `a3f12b9` (PR #214, March 11 2026)
- **What's wrong**: A production OpenAI key was committed in branch `feat/llm-summary`, deleted in a later commit but still reachable through git history. The key has not been rotated.
- **Why it matters**: The key is effectively public to anyone who clones the repo or to anyone who has snapshotted it (GitHub indexers, fork bots). An attacker can drain credits and, more importantly, exfiltrate prompts that may contain user data.
- **Recommended fix**: Rotate the key in the OpenAI dashboard now. Install `gitleaks` as a pre-commit hook. Audit all historical commits for additional secrets with `trufflehog filesystem .`. Move all secrets to Vercel project env vars or a secret manager (1Password, Doppler, AWS Secrets Manager).

## High

### H-1. Analytics and Stripe scripts loaded before EU consent
- **Law**: GDPR Art. 6; ePrivacy Directive (Art. 5(3))
- **Location**: `src/app/layout.tsx:34-58`
- **What's wrong**: `PostHogProvider`, `<Script src="https://js.stripe.com/v3" />`, and the Sentry browser SDK initialize on every page load before any consent UI is shown. EU users are tracked from first visit.
- **Why it matters**: Non-essential trackers in EU/UK require prior opt-in. Recent CNIL, Datatilsynet, and ICO fines (€10M+) target exactly this pattern.
- **Recommended fix**: Move PostHog and the Sentry browser SDK behind a consent gate (e.g. `cookiebot`, `iubenda`, or a custom `ConsentProvider`). Keep Stripe Elements only on payment routes. The banner must offer "Reject all" with equal prominence to "Accept all".

### H-2. No "Do Not Sell or Share My Personal Information" link
- **Law**: CCPA §1798.135; CCPA Regulations §7025 (Global Privacy Control)
- **Location**: `src/components/Footer.tsx:1-60`
- **What's wrong**: The site embeds advertising-grade trackers (PostHog with autocapture, Facebook Pixel referenced in `src/lib/pixels.ts:8`), which qualify as "sharing" under CPRA. No "Do Not Sell or Share" link exists in the footer, and the application does not honor the `Sec-GPC` header.
- **Why it matters**: CPRA-eligible businesses must publish this link and honor the GPC signal. A regulator audit would fail.
- **Recommended fix**: Add a `<Link href="/privacy/do-not-sell">Do Not Sell or Share My Personal Information</Link>` to the global footer. Implement a server-side preference (cookie + `userPreference.shareOptOut`) that gates all sharing trackers. Read `Sec-GPC: 1` in middleware (`src/middleware.ts`) and set the preference automatically.

### H-3. No data export endpoint
- **Law**: GDPR Art. 15, Art. 20; CCPA §1798.110
- **Location**: `src/app/api/account/` (no `export` route)
- **What's wrong**: Users cannot obtain a copy of their data. The account settings page links only to "Edit profile" and "Change password" (Clerk-managed).
- **Why it matters**: Right of access and portability are unsatisfied. CCPA's right to know is unsatisfied.
- **Recommended fix**: Implement `GET /api/account/export` returning a ZIP containing JSON files: `user.json`, `tasks.json`, `comments.json`, `attachments.json`, `ai_conversations.json`, plus a `metadata.json` with categories, sources, recipients, and retention. Re-authenticate before issuing. Expose from the Settings UI.

### H-4. User PII embedded in OpenAI prompts beyond what the feature needs
- **Law**: GDPR Art. 5(1)(c) — data minimization
- **Location**: `src/lib/ai/summarize.ts:21-44`
- **What's wrong**: `summarizeTasks()` constructs the prompt with the full `user` object: `name`, `email`, `phone`, `company`, `country`, `lastIp`. Only `name` is referenced in the system prompt; the rest is included "for context".
- **Why it matters**: OpenAI is a US-hosted processor. Each call exports more PII than necessary. If the user later requests erasure, you must also propagate to OpenAI's content retention.
- **Recommended fix**: Refactor the prompt to include only `user.firstName`. Move PII enrichment to the application layer where it is actually needed. Add a `redactForLLM(user)` helper and unit-test that no disallowed fields leak.

### H-5. Cross-border transfer of EU user data with no documented safeguards
- **Law**: GDPR Art. 44–46
- **Location**: Inferred from Vercel hosting region (`vercel.json:3` → `iad1`) and Supabase project region (`README.md:42` → `us-east-1`)
- **What's wrong**: EU/UK users' data is stored and processed in the US. The privacy policy does not mention transfers, SCCs, or Transfer Impact Assessment.
- **Why it matters**: Post-Schrems II, US transfers require SCCs and a TIA. Hosting in `iad1` is not itself a violation but must be disclosed and contractually safeguarded.
- **Recommended fix**: Sign Standard Contractual Clauses with Vercel and Supabase (both publish standard DPAs covering SCCs). Complete a brief TIA. Either move EU users' data to an EU region (`fra1` on Vercel, `eu-west-1` on Supabase) or disclose the US transfer in the privacy policy.

### H-6. JWT stored in `localStorage` with 30-day expiry
- **Law**: GDPR Art. 32(1)(b)
- **Location**: `src/lib/auth-client.ts:18`, `src/lib/auth-client.ts:34`
- **What's wrong**: A custom JWT issued for a "Public API" feature is stored in `localStorage` and used as a bearer token for `/api/v1/*`. Expiry is 30 days, no refresh-token rotation.
- **Why it matters**: An XSS bug exfiltrates the token for 30 days of unattended access. There is no revocation path.
- **Recommended fix**: Move the token to an `HttpOnly; Secure; SameSite=Lax` cookie. Shorten access-token lifetime to 15 minutes with a rotating refresh token. Add a server-side `tokens` table with revocation capability.

## Medium

### M-1. Privacy policy out of date
- **Law**: GDPR Art. 13; CCPA §1798.130
- **Location**: `public/legal/privacy.mdx:1` (last-updated 2024-11-08); `prisma/schema.prisma` last modified 2026-04-30 with new `Address` and `AiConversation` models.
- **What's wrong**: The schema has gained new PII columns since the policy was last reviewed. The policy does not enumerate `Address` data or AI conversation retention.
- **Recommended fix**: Treat any change to a model with PII as a trigger to update the policy. Add a CI check that fails when a model annotated `@map("pii")` is added without a `public/legal/privacy.mdx` change in the same PR.

### M-2. AI conversations retained indefinitely, not in deletion cascade
- **Law**: GDPR Art. 5(1)(e), Art. 17
- **Location**: `prisma/schema.prisma:120-140` (`AiConversation` model has no `deletedAt`, no retention rule)
- **Recommended fix**: Add a `retentionDays` policy (suggested 90) with a scheduled `pnpm clean:ai` job. Include `AiConversation` in the C-2 deletion cascade.

### M-3. Sensitive PII columns stored in plaintext
- **Law**: GDPR Art. 32; Art. 9 (where applicable)
- **Location**: `prisma/schema.prisma:55` (`User.phone`), `prisma/schema.prisma:62` (`User.governmentId`)
- **Recommended fix**: Apply column-level encryption using `pgcrypto` (`pgp_sym_encrypt(... , current_setting('app.key'))`) or application-level AES-256-GCM with keys in Vercel env vars. Restrict decryption to the application layer. Do not log decrypted values.

### M-4. No CSRF protection on cookie-authenticated state-changing routes
- **Law**: GDPR Art. 32(1)(b)
- **Location**: `src/app/api/account/profile/route.ts:12` (`PATCH`), `src/app/api/team/invite/route.ts:8` (`POST`)
- **Recommended fix**: Enable a CSRF protection scheme: double-submit cookie (token in cookie + `X-CSRF-Token` header) or rely on Clerk session enforcement plus a custom header (`X-Requested-With: fetch`) that is validated server-side. Set `SameSite=Lax` on the session cookie if not already.

### M-5. Global Privacy Control header not read
- **Law**: CCPA Regulations §7025
- **Location**: `src/middleware.ts` (no GPC handling)
- **Recommended fix**: In middleware, read `req.headers.get('Sec-GPC') === '1'` and persist as `userPreference.shareOptOut = true`. Gate all sharing trackers behind that preference.

### M-6. Cookie banner offers no granular categories
- **Law**: ePrivacy Directive Art. 5(3)
- **Location**: `src/components/CookieBanner.tsx:1-90`
- **Recommended fix**: Add three categories (necessary, analytics, marketing) with independent toggles and a "Customize" link. Record per-category consent with a version of the policy.

### M-7. No retention period documented for access logs
- **Law**: GDPR Art. 5(1)(e)
- **Location**: `vercel.json` (log retention defaults to 30 days, undocumented in the policy)
- **Recommended fix**: Document the 30-day retention in the privacy policy. If logs include PII (see C-1 fix), align retention with the privacy policy and configure shorter retention where possible.

## Low

### L-1. No HSTS, CSP, or Permissions-Policy headers
- **Law**: GDPR Art. 32 (defense in depth)
- **Location**: `next.config.mjs:1-30` (no `headers()` override)
- **Recommended fix**: Add a `headers()` function returning `Strict-Transport-Security: max-age=31536000; includeSubDomains`, a starter `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`.

### L-2. No DPO contact in privacy policy
- **Law**: GDPR Art. 37 (where applicable)
- **Location**: `public/legal/privacy.mdx`
- **Recommended fix**: Confirm with legal whether a DPO is required at current scale. If yes (large-scale monitoring or special category data), publish a `dpo@<your-domain>` contact.

### L-3. PII columns not annotated in the schema
- **Law**: best practice for data protection by design (GDPR Art. 25)
- **Location**: `prisma/schema.prisma`
- **Recommended fix**: Add `///` comments to each PII column with category (`identifier`, `contact`, `financial`, `sensitive`). This drives auditing and onboarding.

### L-4. No audit log for admin PII access
- **Law**: GDPR Art. 32; supports Art. 33 breach scoping
- **Location**: `src/app/(admin)/users/[id]/page.tsx:1-110`
- **Recommended fix**: Add a `AdminAccessLog` table with `(adminId, targetUserId, fields, reason, timestamp)`. Log every admin read of user data. Retain 12 months. Restrict the audit log itself to a separate role.

## Passed checks

**Authentication & sessions**
- Password storage delegated to Clerk (argon2id-grade)
- 2FA available (TOTP + WebAuthn)
- Session rotation on login (Clerk-managed)
- HTTPS enforced for all client traffic

**Database & storage**
- Disk-level encryption on Supabase
- Connection over TLS only
- Backups encrypted (Supabase default)
- Row-level security enabled on `User` and `Team` tables

**User rights (partial)**
- Right to rectification: profile edit available in `/(account)/settings`
- Right to object to marketing: opt-out toggle present

**LLM / AI**
- Anthropic and OpenAI providers configured on paid tier (training opt-out by default)
- AI feature labeled "AI-generated summary" at point of output
- "AI can make mistakes — verify important info" notice shown near chat input

**Third parties**
- DPAs signed with Vercel, Supabase, Clerk, Stripe (confirmed by ops 2026-04)

## Not applicable

- Children under 16: ToS prohibits accounts under 18; no targeted product features for minors.
- LGPD: no Portuguese-language UI, no `.br` domain, no BRL currency, no Brazilian markets in marketing roadmap.
- PIPL: no Chinese-language UI, no ICP filing, no Chinese-targeted product features.
- DPDPA: no Indian-language UI, no INR currency, no Indian-targeted product features.

## Next steps

1. **Today** — Rotate the leaked OpenAI key (C-3). Add the secret-scrubbing log filter (C-1).
2. **This sprint** — Implement account deletion endpoint and cascade (C-2). Stand up data export endpoint (H-3).
3. **Before EU launch** — Add cookie consent gate for EU users (H-1). Sign SCCs and disclose transfers (H-5). Minimize OpenAI prompt PII (H-4).
4. **Before California launch** — Add "Do Not Sell or Share" link and GPC handling (H-2, M-5).
5. **Next month** — Encrypt sensitive columns (M-3). Migrate JWT to HttpOnly cookie (H-6). Update privacy policy against current schema (M-1).

---

*This audit is an engineering checklist, not legal counsel. Consult a qualified data protection lawyer for binding decisions on your specific jurisdiction and product.*
