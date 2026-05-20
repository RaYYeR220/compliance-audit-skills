# Mobile Store Compliance Audit

**Generated**: 2026-05-20
**Framework**: React Native 0.74 (bare workflow)
**Target stores**: Apple App Store, Google Play
**App config**: iOS deployment target 15.0; Android targetSdk 33, minSdk 24. Detected: AdMob, Google Sign-In, Stripe, OpenAI SDK, background location.

## Summary

| Risk     | App Store | Google Play | Total |
|----------|-----------|-------------|-------|
| Blocker  | 3         | 3           | 6     |
| High     | 3         | 2           | 5     |
| Medium   | 3         | 1           | 4     |
| Low      | 1         | 1           | 2     |
| **Total**| 10        | 7           | 17    |

## Blocker

### B-1. No Privacy Manifest — required-reason APIs undeclared
- **Store**: App Store
- **Policy**: Apple Privacy Manifest / required-reason API (upload error ITMS-91053)
- **Location**: `ios/Trailmix/` (no `PrivacyInfo.xcprivacy`); `ios/Pods` contains SDKs using `UserDefaults` and file-timestamp APIs
- **What's wrong**: The app target has no `PrivacyInfo.xcprivacy`, but React Native and bundled libraries call required-reason APIs (UserDefaults category `CA92.1`, file timestamp). App Store Connect rejects the binary at upload.
- **Why it's rejected**: Since May 1, 2024, apps using required-reason APIs must declare an approved reason in a privacy manifest.
- **Fix**: Add `ios/Trailmix/PrivacyInfo.xcprivacy` to the target. Declare each required-reason API with an approved reason (UserDefaults → `CA92.1`, file timestamp → `C617.1` if applicable). Update any bundled SDK that lacks its own manifest. Verify approved-reason codes against Apple's current required-reason API documentation.

### B-2. Background location used with no "Always" usage string
- **Store**: App Store
- **Policy**: App Store Review Guideline 5.1.1 (purpose strings)
- **Location**: `src/services/tracking.ts:54` (`requestAlwaysAuthorization`); `ios/Trailmix/Info.plist` (only `NSLocationWhenInUseUsageDescription` present)
- **What's wrong**: The run tracker requests always-on location, but `Info.plist` lacks `NSLocationAlwaysAndWhenInUseUsageDescription`. iOS won't grant the permission and review rejects the build.
- **Fix**: Add `<key>NSLocationAlwaysAndWhenInUseUsageDescription</key><string>Trailmix tracks your route in the background so your run is recorded even when the screen is off.</string>`.

### B-3. Digital subscription sold via Stripe inside the app
- **Store**: App Store
- **Policy**: App Store Review Guideline 3.1.1 (In-App Purchase)
- **Location**: `src/screens/Paywall.tsx:88` (`stripe.confirmPayment(...)` for the "Pro" plan)
- **What's wrong**: The Pro subscription is a digital service consumed in the app but is charged through Stripe, not StoreKit. Outside the narrow US external-link situation, this is a hard rejection.
- **Why it's rejected**: Digital goods/subscriptions must use Apple In-App Purchase unless a qualifying exception or the correct external-purchase entitlement applies.
- **Fix**: Sell the Pro plan via StoreKit IAP (directly or via RevenueCat). If you intend to use external links in the US under the 2025 ruling, verify the present rule and entitlement state per country before relying on it — this area is unsettled.

### B-4. Foreground location service has no `foregroundServiceType`
- **Store**: Google Play
- **Policy**: Android 14+ foreground service requirement
- **Location**: `android/app/src/main/AndroidManifest.xml:41` (`<service android:name=".TrackingService">` with no `foregroundServiceType`); `android/app/src/main/java/.../TrackingService.kt:33` (`startForeground(...)`)
- **What's wrong**: The tracking service starts in the foreground but declares no type. On Android 14+ this throws `MissingForegroundServiceTypeException` and crashes; Play rejects it.
- **Fix**: Add `android:foregroundServiceType="location"` to the `<service>` and the `FOREGROUND_SERVICE_LOCATION` permission. Confirm the use case qualifies under Play's foreground-service policy.

### B-5. Target API level below the Play minimum
- **Store**: Google Play
- **Policy**: Google Play Target API Level Policy
- **Location**: `android/build.gradle:8` (`targetSdkVersion = 33`)
- **What's wrong**: `targetSdk` is 33 (Android 13). New apps and updates must target API 35 (snapshot early 2026), rising to API 36 on August 31, 2026. Play blocks submission.
- **Fix**: Set `targetSdkVersion`/`compileSdkVersion` to the current required level (35 as of this snapshot — verify). Re-test permissions, foreground services, and background limits after bumping. 

### B-6. Digital subscription via Stripe (Android side)
- **Store**: Google Play
- **Policy**: Google Play Payments policy
- **Location**: `src/screens/Paywall.tsx:88` (shared Stripe flow)
- **What's wrong**: Same Stripe digital-subscription flow applies on Android. Outside enrolled alternative-billing regions, digital goods must use Google Play Billing.
- **Fix**: Use Google Play Billing for the Pro plan, or enroll in an approved alternative-billing program for the target region (e.g. US alternative billing, enrollment deadline Jan 28, 2026 — verify). Document the model per region.

## High

### H-1. App Tracking Transparency not implemented while AdMob is present
- **Store**: App Store
- **Policy**: App Store Review Guideline 5.1.2; ATT framework
- **Location**: `src/ads/admob.ts:12` (GoogleMobileAds init at startup); no `ATTrackingManager.requestTrackingAuthorization` anywhere; `ios/Trailmix/Info.plist` has no `NSUserTrackingUsageDescription`
- **What's wrong**: AdMob accesses the IDFA for ads, but the app never shows the ATT prompt and has no tracking usage string.
- **Fix**: Add `expo-tracking-transparency`/`react-native-tracking-transparency`, call the request before initializing AdMob, gate ad personalization on the result, and add a specific `NSUserTrackingUsageDescription` naming the recipient (e.g. "Share with Google to show relevant ads"). Align `NSPrivacyTracking` in the manifest.

### H-2. Account creation with no in-app account deletion
- **Store**: App Store
- **Policy**: App Store Review Guideline 5.1.1(v)
- **Location**: `src/auth/signup.ts:20` (account creation); no deletion path in `src/screens/Settings.tsx`
- **What's wrong**: Users can sign up but cannot delete their account in-app. This is a consistent rejection.
- **Fix**: Add a "Delete Account" action in Settings that triggers full server-side deletion of the account and associated data.

### H-3. Google Sign-In present without Sign in with Apple
- **Store**: App Store
- **Policy**: App Store Review Guideline 4.8 (Login Services)
- **Location**: `src/auth/google.ts:1` (Google Sign-In); no `ASAuthorizationAppleIDProvider` usage; no `com.apple.developer.applesignin` entitlement
- **What's wrong**: A third-party social login is offered without an equivalent privacy-focused option.
- **Fix**: Add Sign in with Apple and its entitlement alongside Google, or adopt another login meeting Apple's criteria.

### H-4. Background location declared alongside ads, no Play declaration
- **Store**: Google Play
- **Policy**: Google Play Background Location policy
- **Location**: `android/app/src/main/AndroidManifest.xml:18` (`ACCESS_BACKGROUND_LOCATION`); AdMob present in `src/ads/admob.ts`
- **What's wrong**: Background location is a restricted permission requiring one core feature and a Play Console declaration. Background location must never be used for ads, and an ads SDK is present in the same app.
- **Fix**: Keep background location strictly for run tracking, never passed to ad SDKs. Complete the Play Console Permissions Declaration with a demo video showing the core feature. Ensure no ad SDK receives location.

### H-5. No web-accessible account deletion URL
- **Store**: Google Play
- **Policy**: Google Play Data deletion policy
- **Location**: app config / Data safety (no deletion URL declared)
- **What's wrong**: Google requires both in-app deletion and a public web URL for deletion requests, declared in Play Console.
- **Fix**: Publish a deletion-request web page reachable without installing the app and declare it in Play Console Data safety. Also add the in-app deletion from H-2.

## Medium

### M-1. Camera usage string is generic
- **Store**: App Store
- **Policy**: App Store Review Guideline 5.1.1
- **Location**: `ios/Trailmix/Info.plist` (`NSCameraUsageDescription` = "This app needs camera access")
- **Fix**: Make it feature-specific, e.g. "Trailmix uses the camera to take your profile photo." Generic strings are rejected.

### M-2. External AI calls with no consent disclosure
- **Store**: App Store (and privacy policy)
- **Policy**: Privacy / 2025–2026 AI-disclosure review practice
- **Location**: `src/ai/coach.ts:27` (sends workout history + notes to OpenAI)
- **Fix**: Show a consent modal naming the AI provider (OpenAI) and the data shared before the first request. Reflect in privacy labels and policy.

### M-3. Encryption export compliance not declared
- **Store**: App Store
- **Policy**: `ITSAppUsesNonExemptEncryption`
- **Location**: `ios/Trailmix/Info.plist` (key absent)
- **Fix**: Add `ITSAppUsesNonExemptEncryption`. Set `false` if only standard encryption (HTTPS/OS crypto) is used; otherwise `true` with export documentation.

### M-4. Cleartext traffic enabled
- **Store**: Google Play
- **Policy**: Security / network config
- **Location**: `android/app/src/main/AndroidManifest.xml:9` (`android:usesCleartextTraffic="true"`)
- **Fix**: Set to `false`. If a specific domain needs HTTP (rare), scope it in `network_security_config.xml`.

## Low

### L-1. `allowBackup` enabled with no extraction rules
- **Store**: Google Play
- **Policy**: Data protection best practice
- **Location**: `android/app/src/main/AndroidManifest.xml:11` (`android:allowBackup="true"`, no `dataExtractionRules`)
- **Fix**: Add `data_extraction_rules.xml` excluding auth tokens and the local DB, or disable backup for sensitive files.

### L-2. Unused queried URL schemes
- **Store**: App Store
- **Policy**: Info.plist hygiene
- **Location**: `ios/Trailmix/Info.plist` (`LSApplicationQueriesSchemes` lists 14 schemes; code references 2)
- **Fix**: Trim the list to schemes the app actually opens (maps, the payment app you integrate with).

## Passed checks

**Permissions**: `NSMicrophoneUsageDescription`, `NSPhotoLibraryUsageDescription` present and specific; Android runtime permission requests gated correctly.
**Build config**: minSdk 24 acceptable; arm64-v8a included; AAB distribution configured.
**Privacy**: Privacy policy URL present in app config and Settings.
**Payments**: Physical merch store correctly uses Stripe (not store billing).
**Tracking**: No SMS/Call Log permissions; no `QUERY_ALL_PACKAGES`.

## Verify in store console

- **App Store privacy nutrition labels**: confirm they declare location, identifiers (IDFA via AdMob), and usage data shared with OpenAI and Google. Detected collection suggests current labels may be incomplete.
- **Play Data Safety form**: confirm it lists Advertising ID, precise location, and data shared with OpenAI/Google.
- **Demo account**: provide reviewer credentials in App Review notes (login wall present).
- **Closed testing (Play)**: if this is a post-2023 personal developer account, complete the required closed test before requesting production access.
- **Age rating / content questionnaires**: complete on both consoles.

## Submission readiness

**Not ready — 6 Blockers.** Fix in this order:
1. Privacy Manifest (B-1) and background-location usage string (B-2) — both block iOS upload/review.
2. Foreground service type (B-4) and target SDK bump (B-5) — both block Play.
3. Resolve the Stripe digital-subscription model on both stores (B-3, B-6) — decide IAP/Play Billing vs an approved alternative-billing region.
4. Then High items (ATT, account deletion, Sign in with Apple, background-location declaration, web deletion URL).

---

*Store policies change and reviewers exercise discretion. This audit predicts common rejection causes; it does not guarantee approval. Verify volatile requirements (target API level, fees, external-payment entitlements) against current official documentation.*
