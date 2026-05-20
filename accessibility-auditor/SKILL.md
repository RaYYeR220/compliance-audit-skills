---
name: accessibility-auditor
description: Audits a web or mobile app's UI code for accessibility barriers and WCAG 2.2 AA conformance, before users (or regulators) hit them. Detects the platform and framework (React, Vue, Angular, Svelte, plain HTML; React Native, Flutter, native iOS/SwiftUI, native Android/Compose) and applies the right checks. Inspects components, forms, images, buttons, links, modals, color and contrast tokens, focus management, and screen-reader semantics. Flags missing alt text and labels, unlabeled controls, poor contrast, keyboard traps and unreachable interactive elements, missing form labels, heading-order problems, missing language attributes, tiny touch targets, missing reduced-motion handling, and inaccessible authentication. Maps findings to WCAG 2.2 success criteria and to the legal regimes that require them (ADA, Section 508, EN 301 549, the EU Accessibility Act in force since June 2025, AODA). Produces a single ACCESSIBILITY_AUDIT.md report graded by user impact (Critical/High/Medium/Low) with file:line citations, the WCAG criterion, and a concrete fix. Use this skill when the user asks to audit accessibility, check a11y, run a WCAG audit, check screen-reader support, verify keyboard navigation, check color contrast, review for ADA or EAA compliance, make the app accessible, check alt text and labels, or prepare for an accessibility review.
---

# Accessibility Auditor

You are auditing a UI codebase for accessibility barriers and WCAG 2.2 Level AA conformance. Your goal is to find concrete, citable issues that exclude users with disabilities (and create legal exposure), and produce a clean, prioritized report.

## Important boundary

This skill audits what is visible in code (static analysis). Some WCAG criteria can only be fully verified by running the app with a screen reader, keyboard, and zoom — list those under "Verify manually" rather than asserting pass/fail. End every report with: *"This is a static code audit, not a full conformance evaluation. Some criteria require manual testing with assistive technology. For a binding conformance claim (VPAT/ACR) or legal certainty, engage an accessibility specialist."*

Target **WCAG 2.2 Level AA** by default — it is the practical and legal baseline. Note where 2.2 adds criteria beyond 2.1, since some regimes (e.g. the EU harmonized standard) still reference 2.1.

## Workflow

Execute these phases in order.

### Phase 1 — Detect platform, framework, and legal scope

1. Identify the platform and framework:
   - **Web**: `package.json` with `react`, `vue`, `@angular/core`, `svelte`, `next`, `nuxt`, `astro`; or plain `.html`/`.css`.
   - **React Native**: `react-native` in `package.json`; `.tsx` using `View`/`Text`/`Pressable`.
   - **Flutter**: `pubspec.yaml`, `.dart` with widgets.
   - **Native iOS**: `.swift` (SwiftUI `View` / UIKit), `.xib`/`.storyboard`.
   - **Native Android**: `.kt`/`.java`, Jetpack Compose (`@Composable`), or XML layouts in `res/layout/`.
2. Load the relevant references:
   - `references/wcag-web.md` for any web framework or HTML.
   - `references/mobile-a11y.md` for React Native, Flutter, native iOS, native Android.
   - `references/legal-standards.md` to set the legal scope.
3. Determine **legal scope** from market signals (so the report cites the right regime): EU presence (locales `de`/`fr`/`es`/`en-GB`, `.eu`/country domains, EUR) → **EU Accessibility Act / EN 301 549**; US (`en-US`, `.com`, public-sector signals) → **ADA / Section 508**; Canada/Ontario → **AODA**. If unclear, default to "WCAG 2.2 AA (the common denominator across ADA, EAA, Section 508)" and note it.

State the detected platform, framework, and legal scope before Phase 2.

### Phase 2 — Discovery

Locate the UI surfaces that carry accessibility obligations. Search for:

1. **Images and media**: `<img>`, `<svg>`, background images used as content, `Image`/`AsyncImage` components, video/audio players. → alt text / labels / captions.
2. **Interactive controls**: `<button>`, `<a>`, `<input>`, custom clickable `<div>`/`<span>` with `onClick`, `Pressable`/`TouchableOpacity`, `GestureDetector`, icon-only buttons. → accessible names, roles, keyboard operability.
3. **Forms**: `<form>`, `<input>`, `<select>`, `<textarea>`, labels, validation/error messages, required fields. → label association, error identification, instructions.
4. **Structure**: headings (`<h1>`–`<h6>`), landmarks (`<nav>`, `<main>`, `<header>`, `<footer>`, `role=`), lists, tables. → semantic structure, heading order.
5. **Color and contrast**: design tokens, theme files, hardcoded colors, color used to convey meaning (status, errors, links). → contrast ratios, not-color-alone.
6. **Focus and keyboard**: `tabindex`, focus traps in modals/dialogs, custom keyboard handlers, skip links, focus-visible styles, autofocus. → keyboard reachability, visible focus, no traps.
7. **Dynamic content**: modals/dialogs, toasts, live regions, accordions, tabs, menus, carousels. → ARIA roles/states, `aria-live`, focus management on open/close.
8. **Motion and timing**: animations, autoplay, carousels, timeouts, `prefers-reduced-motion` handling. → reduced motion, no time limits without control.
9. **Language and metadata (web)**: `<html lang>`, page `<title>`, `lang` on inline language changes.
10. **Touch targets (mobile/responsive)**: control sizes; spacing between adjacent targets.
11. **Authentication**: login flows requiring memorization, CAPTCHAs, OTP retyping. → accessible authentication (WCAG 2.2).

Build the map internally. Do not output it yet.

### Phase 3 — Audit

Apply the loaded references against the discovery map. For each check record pass / fail / needs-manual-verification, with `file:line`, the WCAG 2.2 criterion (e.g. "1.1.1 Non-text Content (A)"), the user impact, severity, and a concrete fix.

**Severity = user impact** (use consistently):

- **Critical** — Completely blocks a group of users from a core task. Examples: icon-only submit button with no accessible name; form input with no programmatic label; keyboard trap in a modal; an interactive element only operable by mouse/touch; an image conveying essential info with no text alternative.
- **High** — Significant barrier; a WCAG A/AA failure on important functionality. Examples: contrast below 4.5:1 on body text; no visible focus indicator; heading structure that misrepresents the page; missing error identification on forms.
- **Medium** — Real barrier with a workaround, or AA failure on secondary content. Examples: non-descriptive link text ("click here"); missing skip link; touch target under 24×24; missing reduced-motion handling.
- **Low** — Best practice or AAA. Examples: enhanced contrast (7:1) not met; minor landmark omissions; redundant ARIA that isn't harmful.

### Phase 4 — Report

Write `ACCESSIBILITY_AUDIT.md` at the repository root (mention overwrite first). Structure:

```markdown
# Accessibility Audit

**Generated**: <ISO date>
**Platform / framework**: <e.g. "Web — Next.js 14 (React)">
**Target standard**: WCAG 2.2 Level AA
**Legal scope**: <e.g. "EU Accessibility Act / EN 301 549; ADA (US)">

## Summary

| Severity | Count |
|----------|-------|
| Critical | N     |
| High     | N     |
| Medium   | N     |
| Low      | N     |
| **Total**| N     |

Conformance snapshot (static): <e.g. "Not conformant — N Level A failures, M Level AA failures detected statically">.

## Critical

### C-1. <Short title>
- **WCAG**: <e.g. "4.1.2 Name, Role, Value (A); 1.1.1 Non-text Content (A)">
- **Impact**: <who is blocked and from what — e.g. "screen-reader users cannot submit the form">
- **Location**: `src/components/SearchBar.tsx:18`
- **What's wrong**: <one or two sentences>
- **Fix**: <concrete code change>

## High
### H-1. ...

## Medium
### M-1. ...

## Low
### L-1. ...

## Verify manually
Checks that need a real screen reader / keyboard / zoom session (e.g. logical reading order, focus order through a complex widget, screen-reader announcement quality, 200%/400% zoom reflow). List each with how to test.

## Passed checks
Terse, grouped (Images, Forms, Structure, Color, Keyboard, Motion).

## Next steps
Priority order, Criticals first.

---

*This is a static code audit, not a full conformance evaluation. Some criteria require manual testing with assistive technology. For a binding conformance claim (VPAT/ACR) or legal certainty, engage an accessibility specialist.*
```

Rules:
- Cite `file:line` and the exact WCAG 2.2 criterion (number, name, level) for every finding.
- Lead each finding with **who is affected and how** — accessibility is about user impact, not abstract rule-breaking.
- Fixes must be concrete and idiomatic for the detected framework. Bad: *"add a label"*. Good: *"Give the icon button an accessible name: `<button aria-label=\"Search\">`, or place visually-hidden text inside it. The bare `<button><SearchIcon /></button>` has no accessible name."*
- Color contrast: when you can read both foreground and background color values, compute the ratio and state it (e.g. "#777 on #fff = 4.48:1, fails 4.5:1 for normal text"). When colors come from variables you can't resolve, route to "Verify manually".
- Do not assert pass on criteria that need runtime checks — put them under "Verify manually".
- Do not fabricate findings.

### Phase 5 — Summary to user

After writing, give a 5-7 line summary: file location, counts by severity, the single most impactful Critical (who it blocks), recommended first fix, and the not-a-full-evaluation reminder. Don't paste the full report.

## Output format example

A good Critical finding:

```markdown
### C-1. Icon-only "Add to cart" buttons have no accessible name
- **WCAG**: 4.1.2 Name, Role, Value (A); 2.4.4 Link Purpose (A)
- **Impact**: Screen-reader users hear only "button" with no indication of what it does or which product it adds — they cannot shop.
- **Location**: `src/components/ProductCard.tsx:54`
- **What's wrong**: `<button onClick={addToCart}><CartIcon /></button>` renders an SVG icon with no text and no `aria-label`, so the accessible name is empty.
- **Fix**: Add a descriptive accessible name including the product:
  `<button onClick={addToCart} aria-label={\`Add ${product.name} to cart\`}><CartIcon aria-hidden=\"true\" /></button>`
  Mark the decorative icon `aria-hidden="true"` so it isn't announced twice.
```

A weak finding to avoid:

```markdown
### Bad — do not do this
- WCAG: accessibility
- Location: the component
- What's wrong: not accessible
- Fix: make it accessible
```

## Edge cases and rules of thumb

- **Component libraries**: If the app uses an accessible component library (Radix, React Aria, MUI, Chakra, Headless UI), many semantics are handled — verify they're used correctly (e.g. `aria-label` still needed on icon buttons) rather than flagging the primitive. Custom components built from `div`s are where most issues live.
- **Design tokens you can't resolve**: When contrast depends on theme variables, compute what you can and otherwise list contrast under "Verify manually" with the token names.
- **Generated/vendor code**: Skip `node_modules`, `build`, `dist`, generated files.
- **Tests/stories**: Findings in Storybook stories or tests are not user-facing; focus on shipped components.
- **WCAG version nuance**: Default to 2.2 AA. If the legal scope is EU/EN 301 549, note that the harmonized standard currently references WCAG 2.1 AA and is being updated to 2.2 — so 2.2-only criteria (e.g. 2.5.8 Target Size, 3.3.8 Accessible Authentication) are advisory there but required under, e.g., updated US guidance. State which criteria are 2.2 additions.
- **Internationalized apps**: Check `lang` is set correctly per locale, not hardcoded to one language.
- **Existing audit file**: If `ACCESSIBILITY_AUDIT.md` exists, ask whether to overwrite or append a dated section.

## What this skill does NOT do

- Does not modify UI code — it only writes the audit report.
- Does not run the app, a browser, a screen reader, or automated a11y tooling (axe, Lighthouse); it complements them with static, framework-aware review and routes runtime-only checks to "Verify manually".
- Does not produce a legally binding VPAT/ACR or certify conformance.
- Does not redesign the UI or generate accessible component implementations beyond targeted fix snippets.
