# Google Play Checklist

Loaded by `SKILL.md` Phase 3 when Android is targeted. Each item: what to look for, the policy cited, rejection-risk severity, and a concrete fix. Verify volatile values (target API level, billing/fees, policy deadlines) at `support.google.com/googleplay/android-developer` and `developer.android.com/google/play/requirements/target-sdk`.

For cross-platform projects, Android config lives in `android/app/src/main/AndroidManifest.xml` and `android/app/build.gradle(.kts)` or is generated from framework config — see `cross-platform.md`.

---

## 1. Target API level — Google Play Target API Level Policy

**Check**: New apps and updates must target a recent API level. **Snapshot (early 2026): target API 35 (Android 15)** for new apps and updates; **API 36 (Android 16) becomes required on August 31, 2026.** Existing apps must target API 34+ to remain installable by new users. Wear OS / Android TV have lower thresholds. **Verify the current minimum before asserting.**

**Look for**: `targetSdkVersion` / `targetSdk` in `android/app/build.gradle(.kts)` (or `android/build.gradle`, or framework config). Also `compileSdk`.

**Red flags**:
- `targetSdk` below the current Play minimum → **Blocker** (Play blocks the submission).
- `compileSdk` lower than `targetSdk` → build inconsistency, **High**.

**Fix**: Set `targetSdk` (and `compileSdk`) to the current required level. Bumping target SDK can change runtime behavior (permissions, foreground services, background limits) — re-test those paths. Verify the present requirement; it increments roughly yearly.

---

## 2. Foreground service types — required Android 14+ (API 34+)

**Check**: Every foreground service must declare a `foregroundServiceType` in the manifest, hold the matching permission, and (for sensitive types) qualify under Play's foreground-service policy. Missing the type crashes the app (`MissingForegroundServiceTypeException`).

**Look for**:
- `<service ... android:foregroundServiceType="...">` in `AndroidManifest.xml`.
- `startForeground(...)` calls in code and the type passed.
- Matching `FOREGROUND_SERVICE_*` permissions (e.g. `FOREGROUND_SERVICE_LOCATION`, `FOREGROUND_SERVICE_DATA_SYNC`, `FOREGROUND_SERVICE_MEDIA_PLAYBACK`).

**Red flags**:
- Foreground service with no declared type → **Blocker** (crash + rejection).
- Type declared but no qualifying use case per Play policy → **High**.
- Android 15 limits: `dataSync` capped at 6 hours/day; `shortService` capped at ~3 minutes and cannot change type. Long-running misuse → **High**.

**Fix**: Declare the correct `foregroundServiceType` and add the matching `FOREGROUND_SERVICE_*` permission. Ensure the use case qualifies. For long background work, prefer `WorkManager` over a persistent foreground service. Play Console requires a declaration justifying sensitive foreground-service types.

---

## 3. Restricted and sensitive permissions

**Check**: Several permissions require a qualifying core use and often a Play Console Permissions Declaration. Requesting them without justification is rejected.

**Look for** these in `AndroidManifest.xml`:

| Permission | Policy | Notes |
|------------|--------|-------|
| `QUERY_ALL_PACKAGES` | Package visibility policy | Only for apps whose core function needs broad app visibility; requires declaration. Prefer `<queries>` for specific packages. |
| `ACCESS_BACKGROUND_LOCATION` | Background location policy | One feature only, core to the app, never for ads; requires declaration + video. |
| `READ_SMS` / `RECEIVE_SMS` / `SEND_SMS` | SMS/Call Log policy | Default handler or narrow exceptions only. |
| `READ_CALL_LOG` / `WRITE_CALL_LOG` / `PROCESS_OUTGOING_CALLS` | SMS/Call Log policy | Default dialer or narrow exceptions only. |
| `MANAGE_EXTERNAL_STORAGE` | All files access policy | Rarely permitted; use scoped storage / SAF instead. |
| `BIND_ACCESSIBILITY_SERVICE` | Accessibility policy | Only genuine accessibility use; misuse is a removal cause. |
| `SCHEDULE_EXACT_ALARM` | Exact alarm policy | Use `USE_EXACT_ALARM` only for alarm/calendar core apps; otherwise inexact alarms. |
| `USE_FULL_SCREEN_INTENT` | Full-screen intent policy | Only for calls/alarms; otherwise declaration required. |
| `READ_MEDIA_IMAGES` / `READ_MEDIA_VIDEO` | Photo/Video permissions policy | Broad media access restricted; use Photo Picker for one-off selection. |
| `READ_CONTACTS` | Contacts policy (effective Oct 28, 2026) | Apps not needing broad access must use the Contact Picker. |

**Red flags**:
- Any restricted permission with no qualifying core feature → **High/Blocker**.
- Broad media or storage permission where the Photo Picker / SAF would suffice → **High**.
- Background location present alongside ad SDKs → **Blocker** (explicitly disallowed for ads).

**Fix**: Remove permissions you don't need. Replace broad access with scoped alternatives (Photo Picker, Storage Access Framework, Contact Picker, specific `<queries>`). For permissions you genuinely need, complete the Play Console Permissions Declaration and prepare a demo video.

---

## 4. Google Play Billing vs alternative billing

**Check**: Digital goods and services historically must use Google Play Billing. This is changing by region.

**Look for**:
- Google Play Billing Library usage (`com.android.billingclient`).
- Third-party payment SDKs (Stripe, PayPal, Braintree, Adyen) used for **digital** goods inside the app.
- Links out to web checkout for digital goods.

**Regional nuance (verify current)**:
- **US**: Following the Epic v. Google injunction, Google announced (Dec 9, 2025) that it will not require Google Play Billing for US apps and will allow other in-app payment methods and out-links. Developers offering alternative billing must enroll and integrate Google's alternative-billing APIs (info screens, parental controls, transaction reporting) by **January 28, 2026**; the revised policy runs until **November 1, 2027**. A service fee may apply.
- **EEA (DMA)**: Alternative billing and external offers programs are available under program terms.
- **Elsewhere**: Google Play Billing generally required for digital goods, with the User Choice Billing pilot in some markets.

**Red flags**:
- Digital goods via a third-party processor in a region where Play Billing is required and no approved alternative-billing enrollment → **Blocker**.
- Physical goods/services forced through Play Billing (the reverse error) → **High** (physical goods must use a normal processor, not Play Billing).

**Fix**: For digital goods, use Google Play Billing unless you are enrolled in an approved alternative-billing program for the target region. For physical goods/services, use a standard processor. Document the model per region and verify the present policy state.

---

## 5. Data Safety section — mandatory and must match behavior

**Check**: Every app must complete an accurate Data Safety form in Play Console describing data collected, used, and shared. The form lives in the console, but the skill can detect collection from code and flag likely mismatches.

**Look for**: SDKs and code that collect identifiers (Advertising ID, Android ID), location, contacts, financial info, messages, photos, usage/analytics. Outbound sharing to analytics/ads/CRM.

**Red flags**: Detected collection/sharing that the Data Safety form likely omits → **High**. Place the actionable item under "Verify in store console" since the form is console-side, but list the detected data types so the user can reconcile.

**Fix**: Provide the detected data-type and recipient list. Instruct the user to ensure the Data Safety form covers each, including the Advertising ID if any ads/analytics SDK is present. Mismatches are a frequent rejection and enforcement cause.

---

## 6. Account deletion — Google Play data deletion policy

**Check**: If the app allows account creation, it must let users request account and data deletion both **in-app** and via a **web URL** (the URL is declared in the Play Console Data deletion section). Deletion must remove data, not just deactivate.

**Look for**: Account creation code; an in-app deletion path; a documented web deletion URL.

**Red flags**:
- Account creation with no in-app deletion → **Blocker**.
- No web-accessible deletion request URL → **High** (required in the Data safety form).

**Fix**: Add an in-app "Delete account" flow and a public web URL for deletion requests reachable without installing the app. Declare the URL in Play Console. Ensure associated data is actually deleted.

---

## 7. Privacy policy — required

**Check**: A privacy policy link is required for apps that handle personal/sensitive data (in practice, almost all), declared in the store listing and often linked in-app.

**Look for**: A privacy policy URL in the app config or codebase; whether sensitive permissions/data are present (which makes it mandatory).

**Red flags**: Sensitive data/permissions present and no privacy policy link → **High**.

**Fix**: Publish a privacy policy and link it in the Play Console listing and in-app. Ensure it reflects the actual data handling (see the companion privacy/compliance auditor).

---

## 8. Cleartext traffic — `usesCleartextTraffic` / network security config

**Check**: Production apps should not permit cleartext (HTTP) traffic.

**Look for**: `android:usesCleartextTraffic="true"` in `AndroidManifest.xml`; a `network_security_config.xml` permitting cleartext for non-localhost domains.

**Red flags**: Cleartext enabled for real domains → **Medium** (security; can draw policy attention). Sending personal data over HTTP → **High**.

**Fix**: Set `usesCleartextTraffic="false"` (default on modern target SDKs). If specific domains need exceptions (rare), scope them narrowly in `network_security_config.xml`.

---

## 9. Backup and data extraction rules

**Check**: Auto-backup can expose sensitive data. Apps handling secrets should scope backups.

**Look for**: `android:allowBackup` in `AndroidManifest.xml`; `data_extraction_rules.xml` (Android 12+) / `fullBackupContent` rules.

**Red flags**: `allowBackup="true"` with no rules while storing tokens/PII → **Medium**.

**Fix**: Define `data_extraction_rules.xml` / backup rules excluding sensitive files (auth tokens, keys, databases with PII), or disable backup for those.

---

## 10. App Bundle and 64-bit

**Check**: New apps must be published as Android App Bundle (`.aab`), and must include 64-bit native libraries (no 32-bit-only).

**Look for**: Build/publish configuration; any 32-bit-only native ABI restriction (`abiFilters` excluding `arm64-v8a`).

**Red flags**: APK-only distribution for a new app → **Blocker** (Play requires AAB). 32-bit-only native libs → **Blocker**.

**Fix**: Publish an AAB. Ensure native libraries include `arm64-v8a` (and `x86_64` for emulator/Chromebook coverage).

---

## 11. Deep links / Android App Links

**Check**: App Links (verified `https` deep links) need a Digital Asset Links file and `autoVerify`. Misconfigured links degrade UX and can be flagged.

**Look for**: `<intent-filter android:autoVerify="true">` with `https` data; a corresponding `/.well-known/assetlinks.json` on the domain (cannot verify the remote file from code — note it).

**Red flags**: `autoVerify` set but assetlinks likely missing (note for verification) → **Low/Medium**.

**Fix**: Host `assetlinks.json` at the domain root and confirm the package name and signing-cert fingerprint match.

---

## 12. Families / children policy — Designed for Families

**Check**: Apps targeting children (or appealing to mixed audiences including children) must comply with the Families policy: appropriate content, no behavioral ads to children, certified ad SDKs, privacy disclosures.

**Look for**: Target-audience signals (content, age gate, marketing); ad SDKs and whether they're in Google's self-certified ads program; analytics on children.

**Red flags**: Child-directed app with non-certified ad SDKs or behavioral ads → **Blocker/High**.

**Fix**: If targeting children, opt into Designed for Families, use only certified ad SDKs in compliant mode, disable behavioral ads for minors, and complete the target-audience and content declarations.

---

## 13. Deceptive behavior and metadata

**Check**: App behavior must match its store description; no hidden features, no functionality that activates only under review-evasion conditions; no impersonation.

**Look for**: Logic that changes behavior based on whether it's under review, locale, or build flags hiding features from reviewers; bundled APK downloads at runtime (dynamic code that changes core behavior).

**Red flags**: Review-evasion branching, runtime download of executable code that materially changes behavior → **Blocker** (and account-level risk).

**Fix**: Remove review-evasion logic. Ensure the shipped behavior matches the listing. Don't fetch and execute code that changes core functionality post-review.

---

## 14. Closed testing requirement (new personal developer accounts)

**Check**: Personal developer accounts created after Nov 13, 2023 must run a closed test (commonly cited: ~12–20 testers, ~14 days) before they can apply for production access. This is an account/console gate, not a code issue, but it surprises many first-time publishers.

**Red flags**: First-time personal-account publisher unaware of the closed-testing gate → note under "Verify in store console".

**Fix**: Plan a closed test with the required testers and duration before requesting production. Verify the current threshold in Play Console.

---

## 15. WebView wrapper / minimum functionality

**Check**: Like Apple 4.2, Google removes low-value apps that are thin web wrappers or lack functionality.

**Look for**: An app whose only content is a full-screen `WebView` of a remote site with no native value.

**Red flags**: Pure WebView wrapper, no native features → **Medium/High**.

**Fix**: Add genuine native functionality (push, offline, device integration) or reconsider distribution.
