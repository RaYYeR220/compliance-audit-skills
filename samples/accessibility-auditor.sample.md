# Accessibility Audit

**Generated**: 2026-05-20
**Platform / framework**: Web — Next.js 14 (React 18, Tailwind CSS)
**Target standard**: WCAG 2.2 Level AA
**Legal scope**: EU Accessibility Act / EN 301 549 (in force June 28, 2025; references WCAG 2.1 AA); ADA (US). Signals: `en-GB`/`de` locales, EUR + USD pricing, EU shipping.

## Summary

| Severity | Count |
|----------|-------|
| Critical | 3     |
| High     | 4     |
| Medium   | 3     |
| Low      | 2     |
| **Total**| 12    |

Conformance snapshot (static): **Not conformant** — 4 Level A failures and 4 Level AA failures detected statically; several criteria need manual verification.

## Critical

### C-1. Icon-only cart and search buttons have no accessible name
- **WCAG**: 4.1.2 Name, Role, Value (A); 2.4.4 Link Purpose (A)
- **Impact**: Screen-reader users (VoiceOver/NVDA/JAWS) hear only "button" — they cannot tell what these do and cannot shop or search.
- **Location**: `components/Header.tsx:42` (search), `components/ProductCard.tsx:55` (add to cart)
- **What's wrong**: `<button onClick={...}><SearchIcon /></button>` and `<button><CartIcon /></button>` render SVGs with no text and no `aria-label`, so the accessible name is empty.
- **Fix**: `<button aria-label="Search"><SearchIcon aria-hidden="true" /></button>` and `<button aria-label={\`Add ${product.name} to cart\`}><CartIcon aria-hidden="true" /></button>`. Hide the decorative icon from the a11y tree.

### C-2. Custom dropdown is mouse-only (keyboard cannot operate it)
- **WCAG**: 2.1.1 Keyboard (A); 4.1.2 Name, Role, Value (A)
- **Impact**: Keyboard and switch users cannot select a country/currency — blocking checkout.
- **Location**: `components/CurrencySelect.tsx:20`
- **What's wrong**: The control is a `<div onClick>` toggling a list of `<div onClick>` items. No `tabIndex`, no roles, no key handlers — unreachable and unoperable by keyboard.
- **Fix**: Use a native `<select>`, or a vetted combobox primitive (Radix/React Aria). If custom: `role="combobox"` + `role="listbox"`/`role="option"`, `tabIndex={0}`, and Arrow/Enter/Escape handling per the ARIA combobox pattern.

### C-3. Checkout form inputs have no programmatic labels
- **WCAG**: 3.3.2 Labels or Instructions (A); 1.3.1 Info and Relationships (A)
- **Impact**: Screen-reader users don't know what each field is; the placeholder vanishes on input, leaving no label.
- **Location**: `app/checkout/page.tsx:88-140`
- **What's wrong**: Fields use `placeholder="Email"` etc. with no `<label>` and no `aria-label`. Placeholder is not a label.
- **Fix**: Add associated labels: `<label htmlFor="email">Email</label><input id="email" autoComplete="email" />`. Keep labels visible; use placeholders only for examples.

## High

### H-1. Body text contrast below the minimum
- **WCAG**: 1.4.3 Contrast (Minimum) (AA)
- **Impact**: Low-vision users can't comfortably read product descriptions and metadata.
- **Location**: `tailwind.config.ts:31` (`gray-450: #949494`) used for body text on white in `components/ProductCard.tsx`
- **What's wrong**: #949494 on #ffffff = **2.9:1**, fails the 4.5:1 minimum for normal text.
- **Fix**: Darken to at least #767676 (4.54:1) or darker for body text. Audit other gray tokens used on white.

### H-2. Global `outline: none` removes the visible focus indicator
- **WCAG**: 2.4.7 Focus Visible (AA)
- **Impact**: Keyboard users lose track of where they are on the page.
- **Location**: `app/globals.css:12` (`*:focus { outline: none; }`)
- **What's wrong**: Focus outlines are removed globally with no replacement.
- **Fix**: Remove the rule or replace with a visible `:focus-visible` ring meeting 1.4.11 (3:1), e.g. `*:focus-visible { outline: 2px solid #1a56db; outline-offset: 2px; }`.

### H-3. Form errors shown by color only, not associated with fields
- **WCAG**: 1.4.1 Use of Color (A); 3.3.1 Error Identification (A)
- **Impact**: Color-blind and screen-reader users don't perceive which field failed or why.
- **Location**: `app/checkout/page.tsx:152`
- **What's wrong**: Invalid fields get a red border only; error text isn't linked via `aria-describedby` and `aria-invalid` isn't set.
- **Fix**: Add visible error text, `aria-invalid="true"`, and `aria-describedby="<error-id>"` on the field; include an icon/text cue beyond color.

### H-4. Zoom disabled via viewport meta
- **WCAG**: 1.4.4 Resize Text (AA)
- **Impact**: Users who pinch-zoom on mobile can't enlarge content.
- **Location**: `app/layout.tsx:18` (`viewport: "width=device-width, maximum-scale=1, user-scalable=no"`)
- **Fix**: Remove `maximum-scale=1` and `user-scalable=no`; allow zoom to at least 200%.

## Medium

### M-1. Non-descriptive link text
- **WCAG**: 2.4.4 Link Purpose (A)
- **Location**: `components/PromoBanner.tsx:14` ("Click here"), `app/blog/page.tsx:33` ("Read more" ×12)
- **Fix**: Make link text describe the destination, or add `aria-label`. Avoid repeated identical link text to different targets.

### M-2. Icon buttons below minimum target size **[WCAG 2.2]**
- **WCAG**: 2.5.8 Target Size (Minimum) (AA) — new in 2.2
- **Location**: `components/Header.tsx:42-60` (32×32 with adjacent 16px-spaced icons)
- **What's wrong**: Some targets are under 24×24 with insufficient spacing. (Advisory under EAA today, which references 2.1; required under a 2.2 target.)
- **Fix**: Make targets ≥24×24 (prefer 44×44 for touch) or add spacing.

### M-3. Carousel autoplays with no reduced-motion handling
- **WCAG**: 2.2.2 Pause, Stop, Hide (A); 2.3.3 Animation from Interactions (AAA)
- **Location**: `components/HeroCarousel.tsx:27`
- **Fix**: Provide a pause control; respect `prefers-reduced-motion` to disable autoplay/animation.

## Low

### L-1. No skip link
- **WCAG**: 2.4.1 Bypass Blocks (A)
- **Location**: `app/layout.tsx`
- **Fix**: Add a "Skip to main content" link as the first focusable element targeting `<main id="main">`.

### L-2. Redundant ARIA on native elements
- **WCAG**: 4.1.2 (hygiene)
- **Location**: `components/Nav.tsx:9` (`<button role="button">`)
- **Fix**: Remove redundant `role="button"` from native `<button>`s.

## Verify manually

These need a real screen reader / keyboard / zoom session:
- **Reading & focus order** through the checkout once C-2/C-3 are fixed (enable VoiceOver, Tab through, confirm logical order).
- **Screen-reader announcement quality** of the cart count update (likely needs `aria-live` — check `components/CartBadge.tsx`).
- **200%/400% zoom reflow** — confirm no content loss or horizontal scroll at 320px width.
- **Contrast of theme tokens** that resolve at runtime (dark mode variants in `tailwind.config.ts`).

## Passed checks

- **Structure**: single `<h1>` per page, landmarks (`<nav>`, `<main>`, `<footer>`) present.
- **Images**: product images have meaningful `alt` from `product.name`.
- **Language**: `<html lang>` set from the locale.
- **Forms**: `autoComplete` tokens present on most fields.

## Next steps

1. Fix the three Criticals first — they block core tasks (search, currency selection, checkout) for keyboard and screen-reader users.
2. Restore visible focus (H-2) and fix body contrast (H-1) — broad impact, low effort.
3. Associate form errors (H-3) and re-enable zoom (H-4).
4. Then Medium items; run axe/Lighthouse in CI as a complement and work the "Verify manually" list with a screen reader.

---

*This is a static code audit, not a full conformance evaluation. Some criteria require manual testing with assistive technology. For a binding conformance claim (VPAT/ACR) or legal certainty, engage an accessibility specialist.*
