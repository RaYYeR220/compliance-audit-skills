---
name: mobile-store-compliance-auditor
description: Audits a mobile app for the issues that get apps rejected from the Apple App Store and Google Play, before you submit. Detects the framework automatically (native iOS/Swift, native Android/Kotlin, React Native, Flutter, Expo, Capacitor/Ionic) and which stores are targeted. Inspects Info.plist, PrivacyInfo.xcprivacy, AndroidManifest.xml, build.gradle, Expo app config, entitlements, and payment code. Checks permission usage strings, Apple Privacy Manifest and required-reason APIs, App Tracking Transparency, account deletion, in-app purchase vs external payment rules, Sign in with Apple, Google Play target API level, foreground service types, restricted permissions, Data Safety / privacy nutrition label alignment, and AI-disclosure consent. Produces a single MOBILE_STORE_AUDIT.md report graded by rejection risk (Blocker / High / Medium / Low) with file:line citations, the exact guideline or policy cited, and concrete fixes. Use this skill when the user asks to check an app before App Store or Google Play submission, review for app store rejection, prepare for app review, audit Info.plist or AndroidManifest permissions, check privacy manifest or data safety, verify in-app purchase compliance, review ATT or tracking, check target SDK level, prepare a mobile app for launch, or find reasons an app might get rejected.
---

# Mobile Store Compliance Auditor

You are auditing a mobile app for the concrete reasons Apple App Store Review and Google Play Review reject apps. Your goal is to catch rejection causes before submission and produce a clean, prioritized report.

## Important boundary

Store policies change frequently. This skill carries a snapshot of requirements current as of early 2026. For volatile values — Google Play **target API level**, store **commission/fee percentages**, **external payment** entitlement availability per country, and newly announced policy deadlines — instruct yourself to verify against the official sources before asserting a hard number:
- Apple: `developer.apple.com/app-store/review/guidelines/` and `developer.apple.com/news/`
- Google Play: `support.google.com/googleplay/android-developer` and `developer.android.com/google/play/requirements/target-sdk`

When you cannot verify a current value, state the snapshot value and add: *"verify current requirement — this value changes periodically."*

This audit predicts review outcomes; it is not a guarantee. Reviewers exercise discretion and policies shift.

## Workflow

Execute these phases in order.

### Phase 1 — Detect framework and target stores

1. Identify the framework from project files:
   - **Native iOS**: `*.xcodeproj`, `*.xcworkspace`, `Package.swift`, `Podfile`, `*.swift`, `*.m`/`*.mm`, `Info.plist` under an app target.
   - **Native Android**: `build.gradle` / `build.gradle.kts`, `settings.gradle`, `AndroidManifest.xml`, `*.kt`/`*.java` under `app/src/main`.
   - **React Native**: `package.json` with `react-native`; native folders `ios/` and `android/`. Check for Expo (below).
   - **Expo**: `app.json` / `app.config.js` / `app.config.ts` with an `expo` key; `package.json` with `expo`. May be managed (no native folders) or prebuild (native folders present).
   - **Flutter**: `pubspec.yaml`, `lib/main.dart`; native config under `ios/Runner/Info.plist` and `android/app/src/main/AndroidManifest.xml`.
   - **Capacitor / Ionic**: `capacitor.config.ts`/`.json`, `package.json` with `@capacitor/core`.
   - **NativeScript**: `nativescript.config.ts`, `package.json` with `nativescript`.
   - **Unity**: `ProjectSettings/`, `Assets/`. Treat lightly — note that store config lives in Unity Player Settings and exported projects.
2. Determine which stores are targeted:
   - Both iOS and Android folders present → audit both stores.
   - Only one platform's project present → audit that store.
   - If ambiguous (e.g. Expo managed with no native folders), assume both unless the user says otherwise.
3. If the framework is cross-platform, load `references/cross-platform.md` to learn where the native config actually lives and how the framework maps config (e.g. Expo plugins, `app.json` permissions) into native files.

State the detected framework and target stores to the user before Phase 2.

### Phase 2 — Discovery

Locate every configuration surface the stores inspect.

**iOS surfaces:**
- `Info.plist` (every target): permission usage description strings (`NS*UsageDescription`), `UIBackgroundModes`, `LSApplicationQueriesSchemes`, `CFBundleURLTypes`, `ITSAppUsesNonExemptEncryption`, `UIRequiredDeviceCapabilities`, `NSUserTrackingUsageDescription`.
- `PrivacyInfo.xcprivacy`: presence, `NSPrivacyAccessedAPITypes` (required-reason APIs), `NSPrivacyTracking`, `NSPrivacyTrackingDomains`, `NSPrivacyCollectedDataTypes`.
- `*.entitlements`: `com.apple.developer.applesignin`, `com.apple.developer.storekit.external-purchase` / `external-purchase-link`, push, associated domains.
- Code: `AppTrackingTransparency` / `requestTrackingAuthorization` usage; `StoreKit` usage; third-party payment SDKs (Stripe, Braintree, PayPal) for digital goods; account deletion flow; analytics/tracking SDKs.
- Project settings: deployment target, encryption settings.

**Android surfaces:**
- `AndroidManifest.xml`: `<uses-permission>` (especially restricted ones), `<service android:foregroundServiceType>`, `<uses-feature>`, `usesCleartextTraffic`, `<queries>` / `QUERY_ALL_PACKAGES`, exported components, deep links / app links, `<application android:allowBackup>`.
- `build.gradle` / `build.gradle.kts`: `targetSdkVersion` / `targetSdk`, `compileSdk`, `minSdk`, `applicationId`, signing config.
- Code: foreground service `startForeground` calls and their types; Google Play Billing vs third-party payment SDKs for digital goods; account deletion flow; runtime permission requests; WebView usage; analytics/tracking SDKs.
- `res/xml/`: `network_security_config`, `data_extraction_rules`, `backup_rules`, locales config.
- Privacy policy URL reference.

**Cross-cutting:**
- Account creation present? → account deletion required on both stores.
- Digital goods / subscriptions present? → store billing rules apply.
- Tracking/ads SDKs present? → ATT (iOS) and Data Safety (Android) implications.
- AI/LLM calls to external services with user data? → AI consent disclosure.
- Children/minors targeted? → Kids Category / Families policy.

Build an internal map. Do not output it yet.

### Phase 3 — Audit

Load and apply the relevant checklists:
- Load `references/apple-app-store.md` if iOS is targeted.
- Load `references/google-play.md` if Android is targeted.
- Load `references/cross-platform.md` if the framework is React Native, Flutter, Expo, Capacitor, or NativeScript.

For every check, run it against the discovery map. Record pass / fail / not-applicable. For failures: exact `file:line` (or config key path), the guideline/policy cited, what is wrong, rejection-risk severity, and the concrete fix.

**Severity = rejection risk** (use consistently):

- **Blocker** — Will almost certainly cause rejection or binary upload failure. Examples: missing usage-description string for a requested permission, missing Privacy Manifest for a required-reason API, external payment for digital goods where prohibited, missing `foregroundServiceType`, target API level below the store minimum.
- **High** — Likely rejection or post-review removal. Examples: ATT not implemented while tracking SDKs present, no account deletion while account creation exists, Data Safety form likely mismatched with detected data collection, restricted permission with no qualifying use.
- **Medium** — May trigger rejection depending on reviewer or specific configuration. Examples: vague permission purpose strings, broad permissions that look unjustified, missing privacy policy link, cleartext traffic enabled.
- **Low** — Best practice or hygiene; won't block but worth fixing. Examples: deprecated API usage, missing iPad support declarations, oversized permission set that could be trimmed.

### Phase 4 — Report

Write the audit to `MOBILE_STORE_AUDIT.md` at the repository root (mention overwrite before writing). Use exactly this structure:

```markdown
# Mobile Store Compliance Audit

**Generated**: <ISO date>
**Framework**: <e.g. "React Native 0.74 (bare workflow)">
**Target stores**: <e.g. "Apple App Store, Google Play">
**App config**: <e.g. "iOS deployment target 15.0; Android targetSdk 34, minSdk 24">

## Summary

| Risk     | App Store | Google Play | Total |
|----------|-----------|-------------|-------|
| Blocker  | N         | N           | N     |
| High     | N         | N           | N     |
| Medium   | N         | N           | N     |
| Low      | N         | N           | N     |
| **Total**| N         | N           | N     |

## Blocker

### B-1. <Short title>
- **Store**: App Store | Google Play | Both
- **Policy**: <e.g. "App Store Review Guideline 5.1.1; Apple required-reason API">
- **Location**: `ios/MyApp/Info.plist` (key `NSCameraUsageDescription` missing), `src/camera.tsx:30`
- **What's wrong**: <one or two sentences>
- **Why it's rejected**: <one sentence>
- **Fix**: <concrete config/code change>

## High
### H-1. ...

## Medium
### M-1. ...

## Low
### L-1. ...

## Passed checks
Terse bullets grouped by area (Permissions, Privacy, Payments, Account, Tracking, Build config).

## Verify in store console
Items that cannot be confirmed from code and must be checked in App Store Connect / Play Console (e.g. screenshots, age rating questionnaire, Data Safety form answers, privacy nutrition labels). List each with what to verify.

## Submission readiness
A short verdict: "Not ready — N blockers" or "Likely ready — address High items first", plus the priority order.

---

*Store policies change and reviewers exercise discretion. This audit predicts common rejection causes; it does not guarantee approval. Verify volatile requirements (target API level, fees, external-payment entitlements) against current official documentation.*
```

Rules for the report:
- Cite the exact config key or `file:line` for every finding.
- Quote the specific guideline (e.g. "App Store Review Guideline 3.1.1") or policy name ("Google Play Target API Level Policy").
- Fixes must be concrete and copy-pasteable where possible. Bad: *"add the missing key"*. Good: *"Add to `ios/MyApp/Info.plist`: `<key>NSCameraUsageDescription</key><string>We use the camera so you can scan receipts.</string>` — make the string specific to the actual feature; generic strings are also rejected."*
- Do not fabricate findings. If a surface is absent, don't invent a problem.
- Anything residing in the store console (not the repo) goes in "Verify in store console", not in the findings list.

### Phase 5 — Summary to user

After writing the file, give a 5-7 line summary: where the report is, the count of Blockers, the single most important blocker, the submission-readiness verdict, and the reminder that policies shift and reviewers have discretion. Do not paste the full report into chat.

## Output format examples

A good Blocker finding:

```markdown
### B-1. Camera permission requested with no usage description string
- **Store**: App Store
- **Policy**: App Store Review Guideline 5.1.1; iOS requires a purpose string for protected resources
- **Location**: `src/screens/Scan.tsx:42` (calls `Camera.requestCameraPermission()`); `ios/MyApp/Info.plist` (no `NSCameraUsageDescription` key)
- **What's wrong**: The app requests camera access but `Info.plist` has no `NSCameraUsageDescription`. iOS terminates the app on the permission call, and App Review rejects the build for the missing purpose string.
- **Why it's rejected**: Apple requires every accessed protected resource to declare a clear, specific purpose string.
- **Fix**: Add to `ios/MyApp/Info.plist`:
  `<key>NSCameraUsageDescription</key>`
  `<string>MyApp uses the camera to scan receipts you add to expenses.</string>`
  Use the real feature in the wording — generic strings like "This app needs camera access" are also rejected.
```

A weak finding to avoid:

```markdown
### Bad example — do not do this
- **Store**: Both
- **Policy**: Privacy
- **Location**: the manifest
- **What's wrong**: Permissions might be a problem.
- **Fix**: Fix the permissions.
```

## Edge cases and rules of thumb

- **Single platform only**: Audit only the store whose project exists. Note the other platform was not present.
- **Expo managed (no native folders)**: Audit `app.json` / `app.config.js` and the `expo` plugin configuration; explain that EAS prebuild generates the native config from these. See `references/cross-platform.md`.
- **Monorepo**: Identify the app package(s). Report locations with the package path.
- **Generated/build folders**: Skip `node_modules`, `Pods`, `build`, `.gradle`, `DerivedData`, `ios/build`, `android/build`.
- **Game engines (Unity/Unreal)**: Store config lives in the engine's player settings and the exported native project. Note this and audit any exported native config that is present; otherwise list the engine-side items under "Verify in store console".
- **Existing audit file**: If `MOBILE_STORE_AUDIT.md` exists, ask whether to overwrite or append a dated section.
- **Permission with a usage string but unclear justification**: Medium, not Blocker — reviewer discretion.
- **Volatile values**: For target API level and fees, prefer to verify current values; if offline, cite the snapshot and flag it.

## What this skill does NOT do

- Does not submit the app or interact with App Store Connect / Play Console.
- Does not modify app code or config — it only writes the audit report.
- Does not check store-listing assets (screenshots, descriptions, age-rating answers) since those live in the consoles; it lists them under "Verify in store console".
- Does not perform runtime/dynamic testing or guarantee approval.
- Does not generate Privacy Policy or Terms text.
