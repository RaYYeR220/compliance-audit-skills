# Cross-Platform Frameworks: Where the Native Config Lives

Loaded by `SKILL.md` Phase 3 when the framework is React Native, Flutter, Expo, Capacitor/Ionic, or NativeScript. The store rules in `apple-app-store.md` and `google-play.md` still apply — this file tells you **where to find or generate** the native config those checklists inspect, plus framework-specific gotchas.

Always audit the actual native files when they exist. When they are generated (Expo managed, Capacitor sync), audit the source config that produces them.

---

## React Native (bare / community CLI)

**Native config locations:**
- iOS Info.plist: `ios/<AppName>/Info.plist`
- iOS entitlements: `ios/<AppName>/<AppName>.entitlements`
- iOS privacy manifest: `ios/<AppName>/PrivacyInfo.xcprivacy` (must be added manually and included in the target)
- Android manifest: `android/app/src/main/AndroidManifest.xml`
- Android build config: `android/app/build.gradle` (look for `targetSdkVersion`/`compileSdkVersion`, often via `android/build.gradle` `ext` block)
- JS deps: `package.json`

**Gotchas:**
- **Privacy manifest**: RN apps frequently miss `PrivacyInfo.xcprivacy`. RN itself and many libraries use required-reason APIs (file timestamp, UserDefaults, boot time). Newer versions of React Native and popular libs ship their own manifests, but verify — an outdated lib without one causes the `ITMS-91053` upload rejection. Check `ios/Pods` for bundled manifests.
- **Permissions**: declared directly in the native `Info.plist` / `AndroidManifest.xml`. Libraries like `react-native-permissions` document which keys to add — confirm the keys actually exist.
- **ATT**: typically via `react-native-tracking-transparency` or `expo-tracking-transparency`. Confirm `NSUserTrackingUsageDescription` exists and the request is called before tracking.
- **Target SDK**: often set via `buildscript ext { targetSdkVersion = ... }` in `android/build.gradle`. Trace the variable.
- **Billing**: `react-native-iap` or RevenueCat usually wraps StoreKit / Play Billing correctly; flag any direct Stripe/PayPal use for digital goods.

---

## Expo (managed and prebuild)

**Config source of truth:** `app.json`, `app.config.js`, or `app.config.ts` (the `expo` object). In **managed** projects there are no `ios/` or `android/` folders — EAS Build runs **prebuild** to generate them from this config and installed config plugins. In **prebuild** (bare) projects the native folders exist and may have manual edits.

**Where things map:**
- iOS Info.plist keys: `expo.ios.infoPlist` (e.g. `NSCameraUsageDescription`).
- iOS entitlements: `expo.ios.entitlements`.
- Android permissions: `expo.android.permissions` (and removals via `expo.android.blockedPermissions`).
- Privacy manifest: `expo.ios.privacyManifests` (e.g. `NSPrivacyAccessedAPITypes`) — Expo SDK 50+ supports declaring this in config.
- Plugins: `expo.plugins` — many native capabilities and their permission strings are configured here (e.g. `expo-location`, `expo-camera`, `expo-tracking-transparency`).

**Gotchas:**
- Audit the **config**, not just generated folders — if you only read a stale `ios/` folder you may miss what prebuild will produce. Prefer `app.config.*` as the source of truth; mention if generated folders are out of sync.
- **ATT**: `expo-tracking-transparency` plugin; confirm the usage string is set in the plugin config or `infoPlist`.
- **Privacy manifest**: confirm `privacyManifests` is populated when required-reason APIs or tracking are present; Expo modules increasingly ship their own.
- **Permissions you don't want**: Expo and some libraries add default permissions; use `blockedPermissions` to strip unused ones (e.g. unneeded `ACCESS_FINE_LOCATION`). Over-broad permissions are a rejection cause.
- **Target SDK**: controlled by the Expo SDK version and `expo-build-properties` plugin (`android.targetSdkVersion`). Check the plugin config; otherwise it follows the Expo SDK default — verify that default meets the current Play minimum.

---

## Flutter

**Native config locations:**
- iOS Info.plist: `ios/Runner/Info.plist`
- iOS entitlements: `ios/Runner/Runner.entitlements`
- iOS privacy manifest: `ios/Runner/PrivacyInfo.xcprivacy` (add manually; ensure it's in the Runner target)
- Android manifest: `android/app/src/main/AndroidManifest.xml`
- Android build config: `android/app/build.gradle` (or `build.gradle.kts`) — `targetSdkVersion`/`targetSdk`, often referencing `flutter.targetSdkVersion`
- Dart deps: `pubspec.yaml`

**Gotchas:**
- **Target SDK**: many Flutter projects leave `targetSdkVersion flutter.targetSdkVersion`, which follows the installed Flutter version's default. Confirm that resolves to the current Play minimum; pin it explicitly if unsure.
- **Permissions**: declared in native files. `permission_handler` requires both the manifest/Info.plist entries and (iOS) preprocessor macros in the Podfile to strip unused permission code — unused permission macros left enabled can cause review questions.
- **Privacy manifest**: Flutter engine and plugins use required-reason APIs; ensure `PrivacyInfo.xcprivacy` exists. Recent Flutter and first-party plugins ship manifests — verify third-party plugins.
- **ATT**: `app_tracking_transparency` package; confirm usage string and pre-tracking request.
- **Billing**: `in_app_purchase` or RevenueCat wraps store billing; flag direct third-party processors for digital goods.

---

## Capacitor / Ionic

**Native config locations:**
- Capacitor config: `capacitor.config.ts` / `capacitor.config.json`
- iOS Info.plist: `ios/App/App/Info.plist`
- iOS privacy manifest: `ios/App/App/PrivacyInfo.xcprivacy` (manual)
- Android manifest: `android/app/src/main/AndroidManifest.xml`
- Android build config: `android/app/build.gradle` and `variables.gradle` (often holds `targetSdkVersion`)
- Web/JS deps: `package.json`

**Gotchas:**
- Native folders are committed and edited directly (Capacitor does not regenerate them destructively), so audit them as-is. `npx cap sync` copies web assets and plugin config but does not overwrite your manifest edits.
- **Target SDK**: commonly in `android/variables.gradle` (`targetSdkVersion = ...`). Trace it there.
- **Permissions**: each Capacitor plugin documents required Info.plist keys / manifest permissions; confirm they are present and that none are declared without a corresponding plugin.
- **Privacy manifest**: add manually for the App target; verify plugins ship their own.
- **Minimum functionality**: Capacitor apps wrap a web app — ensure there is native value (plugins, push, offline) to avoid the Apple 4.2 / Play minimum-functionality rejection for thin wrappers.

---

## NativeScript

**Native config locations:**
- iOS Info.plist: `App_Resources/iOS/Info.plist`
- iOS privacy manifest: `App_Resources/iOS/PrivacyInfo.xcprivacy` (manual)
- Android manifest: `App_Resources/Android/src/main/AndroidManifest.xml`
- Android build: `App_Resources/Android/app.gradle`
- Config: `nativescript.config.ts`, `package.json`

**Gotchas:**
- Config in `App_Resources` is merged into the generated native projects at build. Audit `App_Resources` as the source of truth.
- Confirm target SDK in `app.gradle` meets the current Play minimum.
- Add the privacy manifest manually; verify plugin-bundled manifests.

---

## General cross-platform rules

1. **Source of truth over generated output**: For managed/generated setups (Expo managed, parts of Capacitor sync), audit the config that produces native files; note when committed native folders may be stale.
2. **Plugins add permissions silently**: Many capabilities pull in permissions you didn't write. Cross-check every declared permission against an actual feature; strip the rest (`blockedPermissions` in Expo, manifest removal with `tools:node="remove"` in Android, Podfile macros in Flutter/RN for iOS).
3. **Privacy manifest is the cross-platform trap**: Almost every framework requires a manually added or plugin-provided `PrivacyInfo.xcprivacy`. Missing or incomplete manifests are the top binary-rejection cause for cross-platform iOS apps in 2024–2026. Always verify.
4. **Target SDK indirection**: Cross-platform projects often set target SDK via a variable or the framework default. Resolve the actual value and compare to the current Play minimum.
5. **One JS payment SDK can sink the app**: A `stripe`/`paypal` dependency used for digital goods inside the app is a Blocker on both stores in most regions. Confirm whether purchases are digital (store billing) or physical (normal processor).
