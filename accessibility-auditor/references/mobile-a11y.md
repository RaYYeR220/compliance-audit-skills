# Mobile Accessibility Checklist

Loaded by `SKILL.md` Phase 3 for React Native, Flutter, native iOS (SwiftUI/UIKit), and native Android (Compose/Views). WCAG 2.2 principles apply to mobile too, but the APIs differ. Screen readers: **VoiceOver** (iOS), **TalkBack** (Android). Each item maps to the equivalent WCAG criterion.

The most common mobile barrier is the same as on web: **interactive elements with no accessible name** (icon buttons), and **images with no label**.

---

## Accessible names and labels (WCAG 1.1.1, 4.1.2)

**React Native**
- **Look for**: `Image` without `accessibilityLabel`; `Pressable`/`TouchableOpacity` wrapping only an icon with no label; interactive elements missing `accessible={true}`.
- **Red flags**: icon-only `Pressable` with no `accessibilityLabel` → **Critical**; meaningful `Image` with no label → **High**.
- **Fix**: Add `accessibilityLabel="Search"` and `accessibilityRole="button"` (or `"image"`, `"link"`, `"header"`). Group a composite control with `accessible={true}` on the wrapper so it's announced as one element. Mark decorative images `accessibilityElementsHidden`/`importantForAccessibility="no"`.

**Flutter**
- **Look for**: `IconButton`/`GestureDetector` without a semantic label; `Image` without `semanticLabel`; custom widgets lacking `Semantics`.
- **Red flags**: `GestureDetector` acting as a button with no `Semantics(label:, button: true)` → **Critical**.
- **Fix**: Wrap with `Semantics(label: 'Search', button: true, child: ...)`, or use `Tooltip`/`IconButton(tooltip: 'Search')` which feeds the label. Decorative: `ExcludeSemantics` or `Semantics(excludeSemantics: true)`. Set `Image(semanticLabel: ...)`.

**Native iOS (SwiftUI / UIKit)**
- **Look for**: SwiftUI `Image(systemName:)` in a `Button` with no `.accessibilityLabel`; UIKit controls with no `accessibilityLabel`; `isAccessibilityElement` false on custom views.
- **Red flags**: icon `Button` with no label → **Critical**.
- **Fix**: SwiftUI: `Button { } label: { Image(systemName: "magnifyingglass") }.accessibilityLabel("Search")`. UIKit: set `accessibilityLabel`, `accessibilityTraits = .button`. Decorative: `.accessibilityHidden(true)`.

**Native Android (Compose / Views)**
- **Look for**: `Icon`/`Image` with `contentDescription = null` on meaningful content; `Modifier.clickable` on non-semantic elements; XML `ImageView`/`ImageButton` without `android:contentDescription`.
- **Red flags**: meaningful image with `contentDescription = null`, or clickable with no role/label → **Critical/High**.
- **Fix**: Compose: `Icon(..., contentDescription = "Search")`; for click semantics use `Modifier.semantics { role = Role.Button; contentDescription = ... }` or a real `Button`/`IconButton`. Decorative: `contentDescription = null` is correct only when truly decorative. XML: set `android:contentDescription`, or `android:importantForAccessibility="no"` for decorative.

---

## Touch target size (WCAG 2.5.8 [2.2])

**Guidance**: Apple HIG recommends ≥44×44 pt; Android Material recommends ≥48×48 dp; WCAG 2.2 AA minimum is 24×24 CSS px. Use the platform minimum (44/48) as the target.
- **Look for**: small icon buttons, tightly packed list actions, tiny close "×", small checkboxes.
- **Red flags**: tap targets well below 44/48 with no spacing → **Medium/High**.
- **Fix**: Increase the hit area: RN `hitSlop`; Flutter `IconButton` default 48 / wrap with padding / `MaterialTapTargetSize`; iOS enlarge frame or `contentEdgeInsets`; Android `minWidth`/`minHeight` 48dp.

---

## Dynamic type / font scaling (WCAG 1.4.4)

**Look for**: hardcoded font sizes that ignore the OS text-size setting; `allowFontScaling={false}` (RN); fixed pt sizes not using Dynamic Type (iOS); `sp`-vs-`dp` misuse (Android — text must use `sp`); layouts that clip when text grows.
- **Red flags**: `allowFontScaling={false}` on body text → **High**; Android text sized in `dp` → **High**.
- **Fix**: RN: allow font scaling; test with large text. iOS: use Dynamic Type text styles (`.font(.body)`), support larger accessibility sizes. Android: size text in `sp`; let layouts reflow. Flutter: respect `MediaQuery.textScaler`; avoid fixed-height text containers.

---

## Color and contrast (WCAG 1.4.3, 1.4.11)

- **Look for**: hardcoded colors and theme tokens; text contrast below 4.5:1 (3:1 large); meaning by color alone (status chips, validation).
- **Fix**: Meet 4.5:1 / 3:1; add non-color cues (icon + text). Compute ratios where both colors are known; otherwise route to "Verify manually". Support OS dark mode and high-contrast settings.

---

## Focus order and screen-reader navigation (WCAG 1.3.2, 2.4.3)

- **React Native**: control order with `accessibilityElementsHidden` / `importantForAccessibility`; group with `accessible={true}`; manage focus with `AccessibilityInfo.setAccessibilityFocus` after navigation.
- **Flutter**: `Semantics(sortKey: OrdinalSortKey(...))` for order; `MergeSemantics` to combine; `FocusTraversalGroup` for keyboard/switch order.
- **iOS**: `accessibilityElements` to set order; `.accessibilitySortPriority`; post `.screenChanged`/`.layoutChanged` notifications after major UI changes.
- **Android**: `android:accessibilityTraversalBefore/After`; Compose `Modifier.semantics { traversalIndex = ... }`; announce changes with `LiveRegion`.
- **Red flags**: reading order that doesn't match visual order; focus not moved to new content after navigation → **High** (often manual to confirm — note it).

---

## State and roles (WCAG 4.1.2)

- **Look for**: toggles, tabs, checkboxes, expandable rows whose state isn't exposed.
- **Fix**:
  - RN: `accessibilityState={{ checked, expanded, selected, disabled }}`, `accessibilityRole`.
  - Flutter: `Semantics(toggled:, expanded:, selected:, ...)`.
  - iOS: `accessibilityTraits` (`.selected`), `accessibilityValue`; for switches use `UISwitch` or set value.
  - Android: Compose `Modifier.semantics { stateDescription = ...; toggleableState = ... }`; Views use proper widgets.
- **Red flags**: custom toggle with no state announced → **High**.

---

## Status / live announcements (WCAG 4.1.3)

- **Look for**: async results, toasts, validation, loading completion not announced.
- **Fix**: RN `AccessibilityInfo.announceForAccessibility(...)`; Flutter `SemanticsService.announce(...)` or `liveRegion: true`; iOS `UIAccessibility.post(notification: .announcement, ...)`; Android Compose `Modifier.semantics { liveRegion = LiveRegionMode.Polite }`.
- **Red flags**: error/confirmation shown visually only → **High**.

---

## Forms and inputs (WCAG 1.3.1, 3.3.2)

- **Look for**: text fields without an associated label/`accessibilityLabel`; placeholder-only labels; error messages not linked to the field.
- **Fix**: Provide a programmatic label per field; keep visible labels; associate errors and announce them. iOS: set `accessibilityLabel` on the field; Android: `android:hint` + `labelFor`/`TextInputLayout`; RN/Flutter: explicit labels.
- **Red flags**: input with only a placeholder → **High**.

---

## Motion and gestures (WCAG 2.3.3, 2.5.1, 2.5.7 [2.2])

- **Look for**: parallax/large animations without honoring reduce-motion; functionality requiring multi-finger or path-based gestures with no simple alternative; drag-only interactions.
- **Fix**: Honor reduce-motion: RN `AccessibilityInfo.isReduceMotionEnabled`; iOS `UIAccessibility.isReduceMotionEnabled` / SwiftUI `@Environment(\.accessibilityReduceMotion)`; Android `Settings.Global.ANIMATOR_DURATION_SCALE` / `prefersReducedMotion`; Flutter `MediaQuery.disableAnimations`. Provide tap alternatives to drags and complex gestures.
- **Red flags**: essential action only via drag/complex gesture → **High**.

---

## Accessible authentication (WCAG 3.3.8 [2.2])

- **Look for**: OTP/password fields that block paste; in-app CAPTCHAs; flows requiring memorization/transcription.
- **Fix**: Allow paste and password managers/passkeys; avoid cognitive-test CAPTCHAs; support platform biometric/passkey auth.
- **Red flags**: paste disabled on OTP field → **High**.

---

## Platform settings to honor (cross-cutting)

A quick pass: does the app respect these OS settings?
- Larger text / Dynamic Type
- Bold text / increased contrast
- Reduce motion
- Reduce transparency
- Dark mode
- Screen reader running (avoid behavior that breaks with VoiceOver/TalkBack on)

Ignoring these is a frequent, fixable barrier. Flag hardcoded values that override them.

---

## Manual verification reminder

Several mobile checks are only fully verifiable by running the app with VoiceOver/TalkBack: reading order quality, announcement wording, focus movement after navigation, gesture alternatives, and reflow at large text sizes. Put these under "Verify manually" with concrete steps (e.g. "Enable VoiceOver, swipe through the checkout screen, confirm each control announces a name and role").
