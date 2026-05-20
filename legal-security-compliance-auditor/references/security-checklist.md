# Data Security Checklist

This file is loaded by `SKILL.md` during Phase 3. It maps to GDPR Art. 32 (security of processing) and CCPA §1798.150 (private right of action for breaches caused by lack of reasonable security). Findings here often double-cite a privacy law and a security control.

This is a privacy-focused security checklist, not a full appsec audit. It targets controls that directly affect personal data confidentiality, integrity, and availability.

---

## 1. Password storage

**Check**: Passwords must be hashed with a slow, salted, modern algorithm.

**Acceptable**: `argon2id` (preferred), `bcrypt` with cost ≥ 12, `scrypt`, `pbkdf2-sha256/512` with ≥ 600,000 iterations (OWASP 2024).

**Unacceptable**: `md5`, `sha1`, `sha256` alone, `sha512` alone, plaintext, custom hashing, base64 "encoding", reversible "encryption" of passwords.

**Look for**:
- Imports of `argon2`, `bcrypt`, `bcryptjs`, `scrypt`, `Werkzeug` (Python — must be `pbkdf2_sha256` or `argon2`), `BCrypt::Password` (Ruby), `password_hash()` with `PASSWORD_ARGON2ID` or `PASSWORD_BCRYPT` (PHP), `golang.org/x/crypto/bcrypt`/`argon2`, ASP.NET Identity `PasswordHasher`.
- Calls to `createHash('md5'|'sha1'|'sha256')` in registration/login.
- `crypto.subtle.digest` used for passwords (wrong; that's a hash, not a password hash).
- Symmetric "encryption" of passwords (any pattern with `crypto.createCipher`, `AES`, `Fernet` applied to passwords).

**Red flags**:
```js
// CRITICAL: SHA-256 is not a password hash
const hash = crypto.createHash('sha256').update(password).digest('hex');

// CRITICAL: MD5
hashlib.md5(password.encode()).hexdigest()

// CRITICAL: bcrypt with low cost
bcrypt.hash(password, 4)  // cost 4 is broken

// CRITICAL: password "encryption" (reversible)
const encrypted = aes.encrypt(password, secret);
```

**Severity**: Critical.

**Fix recommendation**: Replace with `argon2id` using defaults from the OWASP Password Storage Cheat Sheet (m=19456 KB, t=2, p=1) or `bcrypt` cost ≥ 12. Migrate existing hashes on next successful login (hash the verified plaintext with the new algorithm and store it; mark the user as migrated).

---

## 2. Password policy

**Check**: NIST SP 800-63B recommends: minimum 8 characters, allow up to at least 64, allow all printable ASCII and Unicode, no composition rules ("must have a number"), no periodic resets, screen against breached password lists.

**Look for**:
- Validation rules for password length and composition.
- Forced expiration logic (`password_changed_at + 90 days`).
- Whether breached-password screening is implemented (`haveibeenpwned` API, `zxcvbn`).

**Red flags**:
- Max length below 64 characters (especially below 20).
- Rejects spaces or specific characters.
- Forces password rotation on a schedule.
- No breached-password check.

**Severity**: Medium.

**Fix recommendation**: Replace composition rules with breached-password screening (`zxcvbn` for strength, `haveibeenpwned` k-anonymity API for breach check). Allow up to 64+ characters. Remove forced rotation.

---

## 3. Authentication flow integrity

**Check**: Login should resist credential stuffing and enumeration. Reset flows should not leak account existence.

**Look for**:
- Rate limiting on login (per IP, per username).
- Account lockout / progressive delays.
- "User not found" vs "wrong password" — these should return the same response.
- Reset token generation (cryptographically random, short-lived, single-use).
- 2FA support (TOTP, WebAuthn, SMS as fallback only).

**Red flags**:
- Login returns different status/messages for "no such user" vs "wrong password".
- No rate limit on login or password reset.
- Reset tokens are predictable (e.g. timestamp-based, sequential).
- Reset tokens never expire or can be reused.

**Severity**: High when enumeration is trivial; Medium for missing rate limit; Critical for predictable reset tokens.

**Fix recommendation**: Return identical responses regardless of which credential is wrong. Apply rate limiting per IP and per identifier (sliding window). Issue reset tokens from a CSPRNG, expire in 15–60 minutes, mark single-use.

---

## 4. Session management

**Check**: Sessions must use secure cookies and proper invalidation.

**Look for**: Cookie attributes wherever sessions are set.

**Required attributes for session cookies**:
- `HttpOnly` — prevents JS access.
- `Secure` — only over HTTPS.
- `SameSite=Lax` (minimum) or `Strict` where possible.
- `Path` set narrowly.
- Reasonable `Max-Age` / `Expires`.

**Red flags**:
- Session cookie missing `HttpOnly` or `Secure`.
- Session tokens stored in `localStorage` and used for authentication (XSS-readable).
- No session rotation after login (session fixation).
- No invalidation on password change or logout (logout sets `set-cookie` to empty but server-side session remains valid).

**Severity**: High for missing HttpOnly/Secure or missing rotation on login.

**Fix recommendation**: Configure session cookies with `HttpOnly; Secure; SameSite=Lax`. Rotate session ID on login and on privilege change. Invalidate server-side on logout, password change, and security events.

---

## 5. JWT security

**Check**: When JWTs are used for authentication, the algorithm, payload, and storage matter.

**Look for**:
- The signing algorithm. `HS256` requires a strong shared secret; `RS256`/`ES256` use asymmetric keys.
- What goes into the payload.
- Token storage on the client.
- Expiration and refresh strategy.

**Red flags**:
- `alg: 'none'` accepted by the verifier.
- Algorithm confusion (HS256 verifier accepting an RS256 token where the attacker controls the "key").
- Long-lived (days/weeks) access tokens with no refresh model.
- PII in the JWT payload — emails, full names, phone numbers, addresses. JWTs are base64 (readable) and often logged or cached.
- JWT stored in `localStorage` (XSS-readable).
- Manual JWT parsing/signing rather than a vetted library.

**Severity**: Critical for `alg: 'none'` or algorithm confusion. High for PII in payload or long-lived tokens. Medium for `localStorage` storage.

**Fix recommendation**: Use a vetted library (`jose`, `jsonwebtoken` with strict `algorithms` whitelist, `python-jose`, `pyjwt` with explicit `algorithms`). Pin the algorithm at the verifier (`{ algorithms: ['RS256'] }`). Keep payload minimal: `sub`, `iat`, `exp`, role claim. Short-lived access tokens (5–15 minutes) plus rotating refresh tokens stored in HttpOnly cookies.

---

## 6. Encryption in transit

**Check**: All traffic with PII must be over TLS. Internal service-to-service traffic too, for any non-loopback link.

**Look for**:
- HTTPS enforcement in framework config (`force_ssl`, `https_only`, HSTS middleware).
- Insecure HTTP endpoints exposed.
- Disabled TLS verification in HTTP clients (`rejectUnauthorized: false`, `verify=False`, `InsecureSkipVerify: true`, `--insecure`).

**Red flags**:
- HTTP listener bound on a public address.
- TLS verification disabled in production HTTP clients.
- No HSTS header.

**Severity**: Critical for disabled TLS verification on production calls handling PII. High for missing HTTPS enforcement.

**Fix recommendation**: Redirect HTTP → HTTPS at edge. Add HSTS (`max-age=31536000; includeSubDomains; preload` once stable). Never disable certificate verification in production.

---

## 7. Encryption at rest

**Check**: Sensitive PII fields should be encrypted at the application or database level. At minimum, disk-level encryption is enabled for the database volume (most managed DBs do this by default — verify).

**Application-level encryption is recommended for**: government IDs (SSN, passport, driver's license), financial account numbers, health information, biometric templates, precise geolocation history, signed legal documents.

**Look for**:
- Encryption libraries: `pgcrypto`, `cloudkms`, `aws-encryption-sdk`, `Tink`, `libsodium`, `@daiyam/aes-gcm`, Postgres TDE, application-side `crypto.createCipheriv('aes-256-gcm', ...)`.
- Whether sensitive columns are encrypted or stored in plaintext.

**Red flags**:
- Sensitive PII stored in plaintext columns.
- "Encryption" using ECB mode (`aes-256-ecb`) — broken pattern.
- Hardcoded encryption keys in code or config files.
- Same key used for encryption and signing.

**Severity**: Critical for plaintext storage of government IDs, financial accounts, or health data. High for hardcoded keys.

**Fix recommendation**: Use AES-256-GCM (authenticated encryption). Store keys in a managed key service (AWS KMS, Google Cloud KMS, Azure Key Vault, HashiCorp Vault). Use envelope encryption for high-volume data. Rotate keys on a schedule. Document the key custody chain.

---

## 8. Secrets in source control

**Check**: API keys, database passwords, JWT signing keys, OAuth client secrets, and similar must not be committed to the repository.

**Look for**:
- Files like `.env`, `.env.production`, `secrets.json`, `credentials.yaml` tracked by git.
- Hardcoded strings matching key patterns: `sk_live_`, `AKIA`, `AIza`, `ghp_`, `xoxb-`, JWT-like strings, long hex/base64 in config.
- `git log` for previously committed secrets even if removed.

**Red flags**: Any production secret visible in the repo or in commit history.

**Severity**: Critical when production secrets are present. High when historical secrets were not rotated after being committed.

**Fix recommendation**: Move all secrets to environment variables or a secret manager. Add a pre-commit hook (`gitleaks`, `trufflehog`, `detect-secrets`). Rotate any secret that was ever committed, even if removed. Add `.env*` to `.gitignore`.

---

## 9. PII in logs

**Check**: Application logs must not contain PII — passwords, full request/response bodies, tokens, email addresses, phone numbers, or document contents.

**Look for** (cross-reference Phase 2 discovery):
- Logger calls that interpolate request bodies, headers, or user objects.
- Middleware that logs the full request.
- Error handlers that include the request body in error reports.
- Third-party error trackers (Sentry, Rollbar, Bugsnag) — these capture exception context which often includes PII unless filtered.

**Red flags**:
```js
logger.info(`User signed up: ${JSON.stringify(req.body)}`);
console.error('Failed login', { email, password });
Sentry.captureException(err, { extra: { user: req.user } });
```

**Severity**: High when PII is logged at info level in production. Critical when passwords or tokens are logged.

**Fix recommendation**: Use structured logging with explicit fields. Maintain a deny-list of fields stripped before logging (`password`, `token`, `secret`, `authorization`, `cookie`, `ssn`, etc.). Configure error trackers with `beforeSend` hooks to scrub PII from event payloads. Document log retention.

---

## 10. SQL injection and ORM safety

**Check**: Database queries must use parameterization, not string concatenation.

**Look for**:
- Raw query strings with interpolated user input (`db.query("SELECT * FROM users WHERE email = '" + email + "'")`).
- ORM escape hatches called unsafely (`Model.raw`, `sequelize.literal`, `Prisma.$queryRawUnsafe`, `SQLAlchemy.text` with concatenation, `ActiveRecord.where(string)` with interpolation).
- Dynamic table/column names from user input.

**Red flags**: Any of the patterns above.

**Severity**: Critical.

**Fix recommendation**: Use parameterized queries everywhere. For ORMs, prefer the safe variants (`Prisma.$queryRaw` tagged template, `text(...).bindparams(...)` in SQLAlchemy, `Model.where(column: value)` in ActiveRecord). Never accept table/column names from user input — map via a whitelist if needed.

---

## 11. XSS protection

**Check**: User-generated content rendered to the browser must be escaped, or rendered as data not code.

**Look for**:
- `dangerouslySetInnerHTML` in React with non-trusted content.
- `v-html` in Vue with non-trusted content.
- `bypassSecurityTrustHtml` in Angular.
- Direct `.innerHTML` assignment with user content.
- Server templates rendering user content without escaping (rare in modern frameworks but possible in Mustache/Handlebars with `{{{ ... }}}`).
- CSP header presence and strictness.

**Red flags**: User-content interpolation in raw HTML sinks. No CSP header at all.

**Severity**: High.

**Fix recommendation**: Avoid raw HTML sinks for user content. If rich text is required, sanitize with `DOMPurify`, `sanitize-html`, or `bleach`. Add a Content-Security-Policy header with `default-src 'self'` and add only the third parties you actually use. Move toward strict CSP with nonces or hashes.

---

## 12. CSRF protection

**Check**: State-changing requests authenticated by cookies need CSRF protection.

**Look for**:
- Framework CSRF middleware (`csurf`, Django CSRF middleware, Rails `protect_from_forgery`, Laravel CSRF middleware).
- `SameSite=Lax|Strict` on session cookies (provides partial protection).
- API surface accepting cookies + form bodies without anti-CSRF tokens.

**Red flags**: State-changing endpoints accepting cookies with no CSRF token and no `SameSite` on the cookie.

**Severity**: High when cookie auth is used and no protection is present.

**Fix recommendation**: Enable framework CSRF middleware. Set `SameSite=Lax` on session cookies. For SPA APIs, use the double-submit cookie pattern or a custom header (`X-Requested-With`) plus CORS allow-listing.

---

## 13. Security headers

**Check**: Standard security headers should be set.

**Look for**:
- `Strict-Transport-Security` (HSTS).
- `Content-Security-Policy`.
- `X-Content-Type-Options: nosniff`.
- `Referrer-Policy: strict-origin-when-cross-origin` or stricter.
- `Permissions-Policy` restricting camera, microphone, geolocation, etc., to those used.
- `Cross-Origin-Opener-Policy: same-origin`.
- `Cross-Origin-Embedder-Policy` if isolation is desired.

**Severity**: Medium.

**Fix recommendation**: Use a headers middleware (`helmet`, `secure_headers`, `django-csp`, etc.) and customize per route as needed.

---

## 14. CORS configuration

**Check**: `Access-Control-Allow-Origin` should not be `*` for authenticated endpoints, and `Access-Control-Allow-Credentials: true` must never combine with `*`.

**Look for**: CORS middleware setup. Wildcard origins combined with credentials.

**Red flags**:
- `Access-Control-Allow-Origin: *` with credentials.
- Origin reflected from request without allow-list (`Access-Control-Allow-Origin: ${req.headers.origin}` with no validation).

**Severity**: High when reflected.

**Fix recommendation**: Maintain an allow-list of origins. Validate the request origin against the list and only echo if it matches.

---

## 15. File upload safety

**Check**: Uploads handling PII or executable content need restrictions on type, size, storage location, and access.

**Look for**:
- Upload endpoints.
- Allowed extension / MIME type checks.
- Whether uploaded files are served from the same origin as the app (XSS risk) or a separate domain.
- Whether content-type is forced server-side (preventing browsers from sniffing).
- Whether PII-containing uploads (ID photos, medical documents) are encrypted at rest with access controls.

**Red flags**:
- No type or size limit.
- Trusting client-supplied MIME or extension.
- Files served from the application origin.
- PII uploads stored in a public bucket.

**Severity**: High for public PII storage. Medium for missing type/size restrictions.

**Fix recommendation**: Validate by content (magic bytes, not just extension). Enforce size limits. Store on a separate origin or with `Content-Disposition: attachment`. For PII uploads, store in a private bucket with signed URLs and short expiry.

---

## 16. Audit logging of PII access

**Check**: Reads and writes of sensitive PII by admins, support staff, or batch jobs should be logged for accountability.

**Look for**: An `audit_log` or `access_log` table, or middleware that records access to sensitive endpoints.

**Severity**: Medium. Required to meet GDPR Art. 32 and to scope a breach.

**Fix recommendation**: Log each admin/support read of sensitive PII with `(actor, target_user, fields, timestamp, reason)`. Retain at least 12 months. Restrict the audit log itself.

---

## 17. Dependency vulnerabilities

**Check**: Out-of-date or vulnerable dependencies handling PII or auth.

**Look for**:
- Whether `npm audit`, `pip-audit`, `bundle audit`, `cargo audit`, `govulncheck`, `composer audit` is run in CI.
- Lockfiles with known CVE-bearing versions of `bcrypt`, `jsonwebtoken`, `cookie`, ORM, web framework.

**Severity**: Medium-High depending on severity of CVEs.

**Fix recommendation**: Enable dependency scanning in CI and fail on high/critical findings. Use Dependabot/Renovate for managed updates. Pin major versions.

---

## 18. Backup encryption and access

**Check**: Database backups containing PII should be encrypted with separate access controls.

**Look for**:
- Backup configuration (cron job, managed service settings).
- Whether backups are stored encrypted.
- Whether backup retention aligns with the privacy policy retention period.

**Severity**: Medium.

**Fix recommendation**: Use the cloud provider's encrypted backup feature with a separate KMS key. Restrict access. Apply lifecycle policy that matches the documented retention.

---

## 19. Rate limiting at scale

**Check**: Endpoints that can enumerate or extract PII (search, autocomplete, public profile, API listing) should be rate-limited.

**Look for**: Rate-limit middleware on PII-exposing routes.

**Severity**: Medium.

**Fix recommendation**: Apply per-IP and per-account rate limits. Add detection for crawler patterns. Lower default page sizes on listing endpoints.

---

## 20. Time-based controls

**Check**: Account lockouts, password reset windows, and session timeouts must be enforced server-side.

**Look for**: Client-side disable patterns ("disable button for 60 seconds") without server enforcement.

**Severity**: Medium.

**Fix recommendation**: Enforce all timing on the server. Client controls are UX only.
