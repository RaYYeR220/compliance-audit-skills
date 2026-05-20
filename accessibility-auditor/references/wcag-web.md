# WCAG 2.2 AA Checklist — Web

Loaded by `SKILL.md` Phase 3 for web frameworks and HTML. Organized by the four WCAG principles (POUR). Each item: the criterion (number, name, level), what to look for, code patterns, severity guidance, and a fix. Default target is **Level AA**. Items marked **[2.2]** are new in WCAG 2.2.

Framework note: React (`jsx-a11y` catches some of these), Vue, Angular, Svelte, and plain HTML share the same underlying DOM semantics. The patterns below use JSX/HTML; translate attribute names as needed (`class` vs `className`, `for` vs `htmlFor`).

---

## Perceivable

### 1.1.1 Non-text Content (A)
**Look for**: `<img>` without `alt`; `<img alt="">` on meaningful images; icon fonts/SVGs conveying meaning; CSS background images used as content; `<input type="image">` without `alt`.
**Red flags**: meaningful image with no/empty alt → **Critical/High**; decorative image with descriptive alt (noise) → **Low**.
**Fix**: Add concise, meaningful `alt` describing purpose. Decorative images: `alt=""` (HTML) and `aria-hidden="true"` on decorative inline SVGs. Functional images (in a link/button): alt describes the action.

### 1.2.x Time-based Media (A/AA)
**Look for**: `<video>`/`<audio>`, players (video.js, Plyr, YouTube embeds). Captions track (`<track kind="captions">`), transcripts, audio descriptions.
**Red flags**: video with audio and no captions → **High**.
**Fix**: Provide captions for prerecorded audio; transcripts for audio-only; audio description for video where visuals carry info.

### 1.3.1 Info and Relationships (A)
**Look for**: layout done with `<div>`s instead of semantic elements; tables for layout; form fields not associated with labels; headings faked with bold text; lists built from `<div>`s.
**Red flags**: data table without `<th>`/scope; visual heading that's a styled `<div>`; grouped radios without `<fieldset>`/`<legend>` → **High**.
**Fix**: Use semantic HTML (`<table><th scope>`, `<ul>/<ol>`, `<h1>`–`<h6>`, `<fieldset><legend>`). Associate labels (below, 3.3.2).

### 1.3.5 Identify Input Purpose (AA)
**Look for**: inputs collecting user info (name, email, address, tel) without `autocomplete` tokens.
**Fix**: Add appropriate `autocomplete` (`name`, `email`, `tel`, `street-address`, `cc-number`, etc.) so assistive tech and autofill work.

### 1.4.1 Use of Color (A)
**Look for**: meaning conveyed by color alone — required fields shown only in red, links distinguished only by color, chart series, status dots without text/icon.
**Red flags**: error state communicated only via red border → **High**.
**Fix**: Add a non-color cue: icon, text label, underline for links, pattern for charts.

### 1.4.3 Contrast (Minimum) (AA)
**Look for**: text color vs background. Normal text needs **4.5:1**; large text (≥24px, or ≥18.66px bold) needs **3:1**. Check theme tokens, hardcoded hex, placeholder text, disabled-but-readable text, text over images/gradients.
**Red flags**: body text below 4.5:1 → **High**. Compute the ratio when both colors are known and state it (e.g. "#949494 on #ffffff = 2.9:1, fails").
**Fix**: Darken/lighten to meet the ratio. When colors are theme variables you can't resolve, route to "Verify manually" with the token names.

### 1.4.4 Resize Text / 1.4.10 Reflow (AA)
**Look for**: fixed `px` font sizes that don't scale; fixed-width containers; `user-scalable=no` or `maximum-scale=1` in the viewport meta (blocks zoom); horizontal scrolling at 320px width.
**Red flags**: `<meta name="viewport" content="...user-scalable=no">` → **High** (blocks pinch-zoom).
**Fix**: Remove zoom restrictions; use relative units (`rem`); ensure layout reflows to a single column at 320px (mostly a manual check — note it).

### 1.4.11 Non-text Contrast (AA)
**Look for**: UI component boundaries and states (input borders, button edges, focus indicators, toggle on/off, icons conveying info) below **3:1** against adjacent colors.
**Fix**: Ensure interactive component boundaries and focus indicators meet 3:1.

### 1.4.12 Text Spacing (AA)
**Look for**: containers that clip or break when line-height/letter/word spacing increases; `!important` fixed line-heights.
**Fix**: Avoid fixed heights on text containers; allow content to grow.

---

## Operable

### 2.1.1 Keyboard (A)
**Look for**: clickable `<div>`/`<span>` with `onClick` but no keyboard handler and no button role; custom widgets (dropdowns, sliders, drag-drop) without key support; `onMouseOver`-only interactions.
**Red flags**: `<div onClick={...}>` acting as a button with no `role`, `tabIndex`, or key handler → **Critical** (keyboard users can't activate it).
**Fix**: Use a real `<button>`/`<a>`. If a custom element is unavoidable: add `role="button"`, `tabIndex={0}`, and handle Enter/Space.

### 2.1.2 No Keyboard Trap (A)
**Look for**: modals/dialogs that trap focus with no Escape; custom widgets that capture Tab and never release.
**Red flags**: focus trap with no way out via keyboard → **Critical**.
**Fix**: Ensure focus can leave (Escape closes dialogs; Tab cycles within and returns). Prefer a vetted dialog primitive.

### 2.4.1 Bypass Blocks (A)
**Look for**: a "Skip to main content" link as the first focusable element; landmark regions.
**Red flags**: large nav with no skip link → **Medium**.
**Fix**: Add a skip link targeting `<main id="main">`; use landmark elements.

### 2.4.2 Page Titled (A)
**Look for**: `<title>` per page/route; in SPAs, whether the title updates on navigation.
**Fix**: Set a unique, descriptive `<title>` per view (e.g. via `next/head`, `react-helmet`, router title hooks).

### 2.4.3 Focus Order (A)
**Look for**: positive `tabIndex` values (>0) that disrupt natural order; DOM order not matching visual order; `autofocus` jumping unexpectedly.
**Red flags**: `tabIndex={5}` etc. → **High**.
**Fix**: Remove positive tabindex; rely on DOM order; only use `tabIndex={0}`/`{-1}`.

### 2.4.4 Link Purpose (A)
**Look for**: "click here", "read more", "learn more" links; icon-only links with no name.
**Fix**: Make link text descriptive, or add `aria-label`. Avoid duplicate ambiguous link texts pointing to different destinations.

### 2.4.6 Headings and Labels (AA)
**Look for**: descriptive headings and labels; not skipping heading levels (h1→h3); multiple/no `<h1>`.
**Red flags**: skipped levels, empty headings → **Medium/High**.
**Fix**: One logical `<h1>`; nest levels in order; descriptive text.

### 2.4.7 Focus Visible (AA)
**Look for**: `outline: none` / `outline: 0` without a replacement focus style; custom components with no `:focus-visible` styling.
**Red flags**: global `*:focus { outline: none }` with no alternative → **High** (keyboard users lose their place).
**Fix**: Provide a clear visible focus indicator (`:focus-visible` outline/ring) meeting 1.4.11 contrast.

### 2.4.11 Focus Not Obscured (Minimum) (AA) **[2.2]**
**Look for**: sticky headers/footers, cookie banners, or chat widgets that cover the focused element when tabbing.
**Fix**: Ensure the focused element isn't fully hidden behind sticky/overlay content (scroll-padding, offset).

### 2.5.7 Dragging Movements (AA) **[2.2]**
**Look for**: functionality that requires dragging (sliders, drag-drop lists, swipe-to-delete) with no single-pointer alternative.
**Fix**: Provide a click/tap alternative (e.g. up/down buttons alongside a draggable slider).

### 2.5.8 Target Size (Minimum) (AA) **[2.2]**
**Look for**: interactive targets smaller than **24×24 CSS px** without sufficient spacing (icon buttons, close "×", tightly packed links).
**Red flags**: 16px icon buttons adjacent to each other → **Medium**.
**Fix**: Make targets ≥24×24, or add spacing so a 24px circle doesn't overlap neighbors.

### 2.2.1 Timing Adjustable / 2.2.2 Pause, Stop, Hide (A)
**Look for**: session timeouts without warning/extension; auto-advancing carousels, marquees, auto-updating content with no pause control.
**Fix**: Let users turn off/adjust/extend time limits; provide pause/stop for moving/auto-updating content.

### 2.3.1 Three Flashes (A)
**Look for**: flashing content/animations.
**Fix**: Avoid content flashing more than 3×/second.

---

## Understandable

### 3.1.1 Language of Page (A)
**Look for**: `<html lang="...">` present and correct; `<html>` with no `lang`.
**Red flags**: missing `lang` → **High** (screen readers mispronounce).
**Fix**: Set `<html lang="en">` (or the right locale); set `lang` on inline foreign-language passages (3.1.2).

### 3.2.1 On Focus / 3.2.2 On Input (A)
**Look for**: focusing or changing a field that triggers navigation or a surprising context change (e.g. auto-submit on select).
**Fix**: Don't cause unexpected context changes on focus/input; require an explicit action.

### 3.2.6 Consistent Help (A) **[2.2]**
**Look for**: help/contact mechanisms placed inconsistently across pages.
**Fix**: Keep help (contact link, chat, FAQ) in a consistent location/order across pages.

### 3.3.1 Error Identification / 3.3.3 Error Suggestion (A/AA)
**Look for**: form validation that shows errors only by color; errors not associated with their field (`aria-describedby`); no suggestion for fixing.
**Red flags**: error text not linked to the input, or only a red border → **High**.
**Fix**: Identify errors in text, associate via `aria-describedby`, set `aria-invalid`, and suggest a correction.

### 3.3.2 Labels or Instructions (A)
**Look for**: inputs without a `<label>` (or `aria-label`/`aria-labelledby`); placeholder used as the only label.
**Red flags**: input labeled only by `placeholder` → **High** (placeholder disappears on input, isn't a label).
**Fix**: Associate a `<label htmlFor="id">` with each control; keep visible labels; placeholders are supplementary only.

### 3.3.7 Redundant Entry (A) **[2.2]**
**Look for**: multi-step flows that ask the user to re-enter the same info already provided in the same process.
**Fix**: Auto-populate or offer previously entered information rather than requiring re-entry.

### 3.3.8 Accessible Authentication (Minimum) (AA) **[2.2]**
**Look for**: login that requires a cognitive function test with no alternative — solving puzzles, transcribing characters, retyping a code from another screen, blocking paste on password/OTP fields.
**Red flags**: password/OTP field with paste disabled; image CAPTCHA with no accessible alternative → **High**.
**Fix**: Allow password managers and paste; offer alternatives to cognitive tests (email link, passkeys, "remember me"); use object-recognition CAPTCHA at most.

---

## Robust

### 4.1.2 Name, Role, Value (A)
**Look for**: custom controls (toggles, tabs, accordions, comboboxes, modals) built from `<div>`s without ARIA roles/states; icon-only buttons with no accessible name; toggle buttons without `aria-pressed`; tabs without `role="tab"`/`aria-selected`; dialogs without `role="dialog"`/`aria-modal`.
**Red flags**: icon button with empty accessible name → **Critical**; custom widget with no role/state → **High**.
**Fix**: Use native elements where possible; otherwise apply the correct ARIA role, name (`aria-label`/`aria-labelledby`), and state (`aria-expanded`, `aria-selected`, `aria-checked`, `aria-pressed`). Follow the ARIA Authoring Practices for the pattern.

### 4.1.3 Status Messages (AA)
**Look for**: toasts, async results, validation summaries, "added to cart" confirmations updated visually but not announced.
**Red flags**: status update with no live region → **High** (screen-reader users miss it).
**Fix**: Use `role="status"`/`aria-live="polite"` (or `assertive` for errors) so changes are announced without moving focus.

---

## ARIA hygiene (cross-cutting)

- **No ARIA is better than bad ARIA**: a wrong `role` overrides native semantics. Flag `role="button"` on an `<a>` that should navigate, or redundant roles (`<button role="button">`).
- **`aria-hidden` on focusable content** hides it from screen readers while it remains tabbable → confusing; flag it.
- **IDs referenced by `aria-labelledby`/`aria-describedby`/`htmlFor` must exist and be unique** — flag dangling references and duplicate IDs.
- **`tabIndex` > 0** is almost always wrong (see 2.4.3).

## Automated-tool note

Tools like axe-core, Lighthouse, and eslint-plugin-jsx-a11y catch a subset of these statically. Recommend running them in CI as a complement, but they miss context (is this alt text *meaningful*? is the focus order *logical*?) — which is where this code-aware review and the "Verify manually" section add value.
