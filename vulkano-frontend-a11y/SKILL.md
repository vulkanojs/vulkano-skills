---
name: vulkano-frontend-a11y
description: Use when adding or reviewing images, navigation, forms, or other interactive elements in this Vulkano framework project's frontend/ — alt text, aria-label/aria-expanded/aria-current, form label/aria-invalid/aria-describedby wiring, focus-visible, and contrast rules.
---

# Frontend Accessibility

## Overview

Every project meets these minimums unless the user explicitly opted out for that area (check the SEO/Analytics/Accessibility table in PROJECT.md § Project requirements — Accessibility is "on" for every area type, including CMS/admin and widgets). If a task touches images, navigation, forms, or an interactive element and minimums aren't met, fix as part of the task — don't skip silently.

## When to use

Any `frontend/` change involving `<img>`, `<nav>`, dropdowns/togglers, `<form>` inputs, or custom interactive elements.

Not for the form's validation-error JS pattern itself — see vulkano-frontend-form, apply both together.

## Images

- Every `<img>` needs a descriptive `alt`. Purely decorative → `alt=""` (empty, never omitted).
- `src` is an absolute path from `public/` (`/img/...`), never `@website/...`/`@admin/...` or a bundled import — see vulkano-frontend-component § Images / static assets.
- Icon-only buttons/links need `aria-label` describing the action, not the icon: `aria-label="Close"`, not `aria-label="X icon"`.

## Navigation

- `<nav>` gets `aria-label` when there's more than one on the page (`aria-label="Main"`, `aria-label="Footer"`).
- Active link gets `aria-current="page"`.
- Dropdowns/togglers (mobile menu, accordion) need `aria-expanded` on the trigger, `aria-controls` pointing at the panel's id.

```html
<button :aria-expanded="isOpen" aria-controls="mobile-menu" @click="isOpen = !isOpen">
  Menu
</button>
<nav id="mobile-menu" aria-label="Main" v-show="isOpen">...</nav>
```

## Forms

Canonical rule for form field a11y wiring — vulkano-frontend-form repeats the same `label`/`aria-invalid`/`aria-describedby` pattern inline in its full-form worked example, don't diverge from this version.

- Every input has a `<label>` wrapping it (implicit association) — placeholder is never a label substitute.
- Invalid fields get `aria-invalid="true"` + `aria-describedby` pointing at the error span's id (that span needs an `id` for this to work) — matches the `fieldErrors` pattern in vulkano-frontend-form.
- Required fields keep the native `required` attribute even though `novalidate` suppresses its browser UI (per vulkano-frontend-form). Not redundant with `formRules`/`V.required(...)` — it stays for the accessibility tree, gives a quick visual signal when inspecting the DOM (devtools shows it directly on the element, no need to open `formRules` to know), and doubles as a check that a field marked required in JS validation is actually marked required in markup (and vice versa) — a mismatch between the two is a bug worth catching.

```html
<label>
  Email <span class="field-required">*</span>
  <input
    id="email"
    type="email"
    required
    :aria-invalid="!!fieldErrors.email"
    aria-describedby="email-error"
  />
</label>
<span v-if="fieldErrors.email" id="email-error">{{ fieldErrors.email }}</span>
```

Still keep `id` on the input — needed as the `aria-describedby` target, unrelated to the label link. Same pattern for checkbox/radio:

```html
<label class="checkbox">
  <input
    type="checkbox"
    v-model="form.acceptTerms"
    required
    :aria-invalid="!!fieldErrors.acceptTerms"
  />
  I accept the terms <span class="field-required">*</span>
</label>
```

Alternative — explicit `<label for="...">` matched by input `id`, no wrapping: also valid, use it when the layout needs label and input as separate grid items (§ Layout in vulkano-frontend-css) and wrapping would fight that structure:

```html
<label for="email">Email <span class="field-required">*</span></label>
<input
  id="email"
  type="email"
  required
  :aria-invalid="!!fieldErrors.email"
  aria-describedby="email-error"
/>
<span v-if="fieldErrors.email" id="email-error">{{ fieldErrors.email }}</span>
```

Text sits next to the box either way, so wrapping instead of `for`/`id` skips a throwaway `id` and keeps label+control as one grid/flex item.

## Focus and interaction

- Never `outline: none` without a `:focus-visible` replacement — keyboard users must see focus.
- Interactive elements are real `<button>`/`<a>`, never a `<div>`/`<span>` with a click handler and no keyboard support.

## Color and contrast

- Reuse existing design tokens (the project's own danger/error/etc. color variables, per vulkano-frontend-form § required-field asterisk) — don't introduce colors under 4.5:1 text contrast (3:1 for large text/UI components).
- Never convey state (error/required/active) through color alone — pair with text, icon, or `aria-*`.

## Structure

- Heading levels (`h1`-`h6`) follow document order, no skipped levels for styling.
- Use landmark elements (`<header>`, `<nav>`, `<main>`, `<footer>`) over generic `<div>`s where applicable.

## After writing

- Visually verify with a screen reader or the browser's accessibility tree inspector when the change is non-trivial (custom widget, dropdown).
- Run `vp check` and `vp test`.

## Reference

`references/AGENTS/ACCESSIBILITY.md` (full detail).
