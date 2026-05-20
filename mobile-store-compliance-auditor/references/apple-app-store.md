# Apple App Store Checklist

Loaded by `SKILL.md` Phase 3 when iOS is targeted. Each item: what to look for, the guideline cited, rejection-risk severity, and a concrete fix. Guideline numbers refer to the App Store Review Guidelines (verify wording at `developer.apple.com/app-store/review/guidelines/`).

For cross-platform projects, the iOS config still lives in native files (`ios/.../Info.plist`, `ios/.../*.entitlements`, `PrivacyInfo.xcprivacy`) or is generated from framework config — see `cross-platform.md`.

---

## 1. Permission usage description strings — Guideline 5.1.1

**Check**: Every protected resource the app accesses must have a specific purpose string in `Info.plist`. A missing string crashes the app on the permission call and is rejected.

**Look for**: Code that requests or uses a protected resource, then confirm the matching key exists in `Info.plist`:

| Resource | Required Info.plist key |
|----------|------------------------|
| Camera | `NSCameraUsageDescription` |
| Microphone | `NSMicrophoneUsageDescription` |
| Photo library (read) | `NSPhotoLibraryUsageDescription` |
| Photo library (add) | `NSPhotoLibraryAddUsageDescription` |
| Location when in use | `NSLocationWhenInUseUsageDescription` |
| Location always | `NSLocationAlwaysAndWhenInUseUsageDescription` |
| Contacts | `NSContactsUsageDescription` |
| Calendars | `NSCalendarsUsageDescription` (or `NSCalendarsFullAccessUsageDescription` on iOS 17+) |
| Reminders | `NSRemindersUsageDescription` |
| Bluetooth | `NSBluetoothAlwaysUsageDescription` |
| Motion | `NSMotionUsageDescription` |
| Face ID | `NSFaceIDUsageDescription` |
| Speech recognition | `NSSpeechRecognitionUsageDescription` |
| Local network | `NSLocalNetworkUsageDescription` |
| Health | `NSHealthShareUsageDescription` / `NSHealthUpdateUsageDescription` |
| Tracking (ATT) | `NSUserTrackingUsageDescription` |

**Red flags**:
- Code calls a permission API but the key is absent → **Blocker**.
- Key present but the value is generic ("This app needs access") → **Medium** (Apple rejects vague strings).
- Key present for a resource the app never uses → **Low** (remove it; reviewers flag unused sensitive declarations).

**Fix**: Add the key with a specific, feature-referencing string. Example:
`<key>NSCameraUsageDescription</key>`
`<string>MyApp uses the camera to scan receipts you add to expenses.</string>`

---

## 2. Privacy Manifest and required-reason APIs — most common 2024–2026 binary rejection

**Check**: Apps and many third-party SDKs must include `PrivacyInfo.xcprivacy`. Apps that call "required-reason APIs" without declaring an approved reason are rejected at upload (error `ITMS-91053`) since May 1, 2024.

**Look for**:
- Presence of `PrivacyInfo.xcprivacy` in the app target (and in bundled SDKs).
- `NSPrivacyAccessedAPITypes` entries with a category and an approved reason code.
- Required-reason API categories that the code (or an SDK) is likely to touch:
  - **File timestamp** (`NSPrivacyAccessedAPICategoryFileTimestamp`) — `stat`, `fstat`, `getattrlist`, `NSFileManager` attributes, `.creationDate`, `.modificationDate`.
  - **System boot time** (`...CategorySystemBootTime`) — `systemUptime`, `mach_absolute_time`.
  - **Disk space** (`...CategoryDiskSpace`) — `volumeAvailableCapacity`, `NSFileSystemFreeSize`.
  - **Active keyboard** (`...CategoryActiveKeyboards`) — `activeInputModes`.
  - **User defaults** (`...CategoryUserDefaults`) — `UserDefaults` / `NSUserDefaults`.

**Red flags**:
- App or SDK uses any required-reason API and no `PrivacyInfo.xcprivacy` declares it → **Blocker** (upload fails).
- Privacy manifest missing entirely while the app collects data or uses tracking → **Blocker/High**.
- `NSPrivacyTracking` is `true` but `NSPrivacyTrackingDomains` is empty, or vice versa → **High**.

**Fix**: Add `PrivacyInfo.xcprivacy` (Xcode 15+: File → New → App Privacy File). Declare each required-reason API with an approved reason. Example for `UserDefaults`:
```xml
<key>NSPrivacyAccessedAPITypes</key>
<array>
  <dict>
    <key>NSPrivacyAccessedAPIType</key>
    <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
    <key>NSPrivacyAccessedAPITypeReasons</key>
    <array><string>CA92.1</string></array>
  </dict>
</array>
```
Confirm bundled SDKs ship their own manifests; update SDKs that don't. Verify the current approved-reason codes at `developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api`.

---

## 3. App Tracking Transparency — Guideline 5.1.2

**Check**: If the app tracks users across apps/websites owned by other companies (or shares device identifiers like IDFA with data brokers), it must request permission via the ATT framework before tracking, and have `NSUserTrackingUsageDescription`.

**Look for**:
- Tracking/ads SDKs: Meta/Facebook SDK, Google AdMob/GoogleMobileAds, AppLovin, Unity Ads, IronSource, branch, Adjust, AppsFlyer, Singular, TikTok SDK.
- IDFA access (`ASIdentifierManager`, `advertisingIdentifier`).
- Whether `ATTrackingManager.requestTrackingAuthorization` is called before tracking starts.
- `NSUserTrackingUsageDescription` presence.

**Red flags**:
- Tracking SDK present, no ATT prompt → **High** (rejection under 5.1.2; also fraud risk).
- IDFA accessed before ATT authorization → **High**.
- ATT description is generic; as of 2025 Apple expects specific recipients ("Share with Meta for advertising") rather than "for a better ad experience" → **Medium**.
- `NSPrivacyTracking` in the privacy manifest inconsistent with ATT usage → **High**.

**Fix**: Call `ATTrackingManager.requestTrackingAuthorization` before any tracking/IDFA use. Gate tracking SDK initialization on the authorization result. Write a specific `NSUserTrackingUsageDescription`. Ensure the privacy manifest's `NSPrivacyTracking`/`NSPrivacyTrackingDomains` match.

---

## 4. Privacy nutrition labels alignment — Guideline 5.1.1

**Check**: The privacy labels declared in App Store Connect must match the data the app actually collects. The skill can detect collection from code; the label answers live in the console.

**Look for**: Analytics, crash reporting, ads, and backend SDKs that collect identifiers, contact info, location, usage data. Map detected collection to data categories.

**Red flags**: Detected data collection that the user likely did not declare → **High**, but place the actionable verification under "Verify in store console" since the labels themselves are console-side.

**Fix**: List detected data types and recipients. Instruct the user to confirm App Store Connect privacy labels cover all of them. A mismatch is a frequent rejection and post-launch removal cause.

---

## 5. Account deletion — Guideline 5.1.1(v)

**Check**: If the app supports account creation, it must offer in-app account deletion (not just deactivation, not "email us"). A visible path is expected.

**Look for**:
- Account/signup code (`createUser`, `signUp`, auth SDKs).
- A deletion path in settings (`deleteAccount`, `DELETE /account`, a "Delete Account" screen).

**Red flags**:
- Account creation exists, no in-app deletion → **Blocker** (consistently rejected since 2022/2023).
- "Delete account" only emails support or opens a web page that just contacts support → **High**.

**Fix**: Add an in-app "Delete Account" action in settings that initiates full deletion of the account and associated data (server-side). If deletion is regulated (e.g. finance) and must be delayed, disclose the process and timeline in-app.

---

## 6. In-app purchase vs external payment — Guideline 3.1.1 / 3.1.3

**Check**: Digital goods and services consumed in the app must use Apple In-App Purchase, with specific exceptions. Steering users to external payment for digital goods is the classic rejection. Note the evolving US situation (below).

**Look for**:
- StoreKit usage (`SKPaymentQueue`, `Product.purchase`, RevenueCat, Glassfy).
- Third-party payment SDKs (Stripe, Braintree, PayPal, Paddle) used for **digital** goods/subscriptions inside the app.
- Links out to a website checkout for digital goods.
- "Reader" apps, physical goods, or person-to-person services (legitimate non-IAP cases).

**Red flags**:
- Digital subscription sold via Stripe inside the app with no IAP, targeting markets where this is disallowed → **Blocker**.
- External purchase link without the corresponding entitlement where one is required → **Blocker**.
- Physical goods using IAP (the reverse error — physical goods must NOT use IAP) → **High**.

**US / regional nuance (verify current)**: After the April 30, 2025 Epic v. Apple ruling, US apps may include external purchase links, and as of that ruling Apple was barred from commission on them; a December 2025 appeals decision allows Apple a "reasonable" fee and the matter is unsettled (heading to higher courts). Outside the US, linking out for digital goods generally still requires the **StoreKit External Purchase Link Entitlement**, applied for per country. EU DMA adds alternative options. **Verify the present state before declaring a hard rule** — this is the most volatile area in store policy.

**Fix**: For digital goods, use StoreKit IAP unless you qualify for an exception or have the correct external-purchase entitlement for each target country. For physical goods/services, use a normal payment processor and do NOT use IAP. Document which model applies per region.

---

## 7. Sign in with Apple — Guideline 4.8 (Login Services)

**Check**: If the app offers third-party or social login (Google, Facebook, etc.) as the primary login, it must also offer an equivalent privacy-focused option. Sign in with Apple satisfies this; so do certain other options meeting Apple's privacy criteria.

**Look for**:
- Social login SDKs (Google Sign-In, Facebook Login, Twitter/X, LINE, WeChat).
- Presence of Sign in with Apple (`ASAuthorizationAppleIDProvider`, `com.apple.developer.applesignin` entitlement) or another qualifying private login.

**Red flags**: Social login present, no Sign in with Apple or other qualifying option → **High** (rejection under 4.8). Exceptions: apps using only their own account system, education/enterprise apps, or apps using a government/industry-specific identity.

**Fix**: Add Sign in with Apple (and the entitlement) alongside the social options, or adopt another login that meets Apple's criteria (limits data to name/email, allows hiding email, no tracking without consent).

---

## 8. Encryption export compliance — `ITSAppUsesNonExemptEncryption`

**Check**: Apps must declare whether they use non-exempt encryption. Missing this prompts a question on every submission and can stall review.

**Look for**: `ITSAppUsesNonExemptEncryption` key in `Info.plist`.

**Red flags**: Key absent → **Low/Medium** (causes a manual question each submission). Custom/proprietary cryptography (not standard HTTPS) without proper export documentation → **High**.

**Fix**: Add `ITSAppUsesNonExemptEncryption`. Set `false` if the app only uses standard encryption exempt from documentation (e.g. HTTPS, standard OS crypto). If it uses non-exempt encryption, set `true` and prepare export-compliance documentation.

---

## 9. App completeness — Guideline 2.1

**Check**: No crashes, no placeholder content, no broken features, no obvious bugs. Placeholder content ("Lorem Ipsum", "TODO", test data, dead buttons) is a leading rejection.

**Look for**:
- Placeholder strings in shipped UI: `Lorem ipsum`, `TODO`, `FIXME`, `test@test.com`, `Coming soon`, dummy images.
- Hardcoded staging/localhost endpoints that won't work in review.
- Features behind credentials with no demo account provided (note: provide a demo account in App Review notes — console-side).

**Red flags**: Placeholder content in user-facing screens → **High**. Pointing the production build at `localhost`/staging → **High**.

**Fix**: Remove placeholder content. Ensure the reviewed build hits production endpoints. Provide a demo account and any required steps in App Review notes.

---

## 10. Minimum functionality / web wrappers — Guideline 4.2

**Check**: The app must offer native value, not merely repackage a website or be a thin wrapper.

**Look for**: An app whose root is a single full-screen `WKWebView` loading a remote site with little or no native functionality.

**Red flags**: Pure WebView wrapper with no native features, push, or offline value → **High** (4.2 rejection).

**Fix**: Add genuine native functionality (push notifications, offline support, device integrations, native navigation). If the product is inherently web, reconsider whether it belongs in the App Store.

---

## 11. Background modes justification — Guideline 2.5.4

**Check**: `UIBackgroundModes` must match actual functionality. Declaring background location/audio without using it is rejected.

**Look for**: `UIBackgroundModes` entries (`location`, `audio`, `voip`, `fetch`, `processing`, `bluetooth-central`) vs whether the code actually uses each.

**Red flags**: Background mode declared but unused, or used only to keep the app alive → **High**.

**Fix**: Remove unused background modes. For each kept mode, ensure there is a genuine, user-visible feature using it.

---

## 12. AI / external-model data disclosure

**Check**: Apps sending personal data to external AI services are increasingly expected to disclose the provider and data types and obtain consent before sharing. (Reinforced by 2025–2026 review practice and overlaps with privacy guidelines.)

**Look for**: Calls to OpenAI/Anthropic/Google AI and similar with user content; whether a consent screen names the provider and the data shared.

**Red flags**: User content sent to an external model with no disclosure/consent → **High**.

**Fix**: Add a consent modal that names the AI provider(s) and the data types shared, shown before the first send. Reflect this in privacy labels and policy. (See the companion privacy/compliance auditor for the legal dimension.)

---

## 13. Push notifications — Guideline 4.5.4 / 5.1.1

**Check**: Push must be optional; apps must not require enabling push to function, and must not use push for advertising without consent.

**Look for**: Registration for remote notifications; whether the app blocks usage until notifications are enabled; promotional push without an opt-in.

**Red flags**: App unusable unless push is enabled → **High**. Promotional push with no consent → **Medium**.

**Fix**: Make push optional. Gate promotional notifications behind explicit opt-in.

---

## 14. Data minimization / login wall — Guideline 5.1.1(i)

**Check**: Apps may not require users to register or provide personal information unless it is directly relevant to the core functionality, or required by law. Free features should not sit behind an unnecessary account wall.

**Look for**: A forced login/registration before any functionality, where the core feature doesn't require an account.

**Red flags**: Mandatory account for features that don't need one → **Medium/High**.

**Fix**: Allow access to functionality that doesn't require an account without forcing registration. Only gate features that genuinely need identity.

---

## 15. Private API and deprecated symbols — Guideline 2.5.1

**Check**: Use only public APIs. Private/undocumented API usage is rejected at review or by static analysis.

**Look for**: Use of underscore-prefixed selectors via runtime, `valueForKey` into private properties, swizzling system internals, undocumented frameworks.

**Red flags**: Detected private API usage → **High/Blocker**.

**Fix**: Replace with public APIs. Remove any code that calls into private frameworks.

---

## 16. URL schemes and queried schemes — Info.plist

**Check**: `LSApplicationQueriesSchemes` should list only schemes the app actually queries; custom URL types should not collide with others.

**Look for**: `LSApplicationQueriesSchemes` and `CFBundleURLTypes`. Over-broad scheme querying (fingerprinting installed apps) draws scrutiny.

**Red flags**: Large list of queried schemes unrelated to functionality → **Medium**.

**Fix**: Trim queried schemes to those genuinely used (e.g. opening a maps or payment app you integrate with).
