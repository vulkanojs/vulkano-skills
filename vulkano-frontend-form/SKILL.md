---
name: vulkano-frontend-form
description: Use when writing or editing a `<form>` or form view (CRUD create/edit, login, contact) in this Vulkano project's frontend/ — required-field asterisks, useFormValidation rules, fieldErrors, input types, date pickers.
---

# Frontend Form

## Overview

Forms never rely on native browser validation UI (`required`/`:invalid` styling, error bubbles) — it can't be styled consistently and breaks the design system. Validation is always hand-rolled in JS: a rules object per field, checked by the shared `useFormValidation` composable, error state exposed as `fieldErrors`, message rendered inline.

`useFormValidation` is a project-owned, dependency-free composable — not a third-party validation library. If a validation library is added to the project later, swap the composable's internals only; every form's `formRules`/`validate(...)` usage stays unchanged.

## When to use

Any `<form>` add/edit in `frontend/` — contact forms, login, CRUD create/edit views, even a quick one-field form.

MANDATORY: load this before writing or editing any `<form>`; never write form markup/logic from memory of a past form — conventions here may have moved.

Not for component file layout — see vulkano-frontend-component.

## Required-field pattern

- Validation runs through `const { fieldErrors, validate, clearError } = useFormValidation(formRules)` and `validate(form)` — synchronous, returns a boolean (see Skeleton). Never native `required`/`:invalid` UI.
- `formRules` is a plain object: `{ email: [V.required('...'), V.email('...')] }` — each entry an array of validator functions from `frontend/<entrypoint>/utils/validators.js` (`V.required(message)`, `V.email(message)`, ...), checked in order, first failing rule wins.
- `fieldErrors` is the reactive object returned by `useFormValidation` — never re-declared locally.
- Every required field's label gets a red asterisk: reuse a shared `.field-required` (or equivalent BEM element) — both ship in the template: `.field-required` in `scss/_base.scss`, colored with `var(--color-danger)` from `scss/_tokens.scss`. Never hardcode a red per view.
- Error message rendered inline below the input: `<span class="*__field-error">{{ fieldErrors.email }}</span>`.
- Invalid input gets a `*__input--invalid` class for the red border.
- Required inputs keep the native `required` attribute even though `novalidate` suppresses its browser UI — not redundant with `V.required(...)` in `formRules`: it stays for the accessibility tree, gives a quick visual signal in devtools without opening `formRules`, and a mismatch between the two (required in markup but not in rules, or vice versa) is a bug worth catching.
- No `frontend/<entrypoint>/views/Login/` exists in a fresh scaffold — it's not a file to go open and copy. Follow the `useFormValidation` + `formRules` + `fieldErrors` + `<span class="*__field-error">` + `*__input--invalid` shape from the Skeleton below instead; once a project's first login/form view exists, treat _that_ as the local reference for the next one.
- `composables/useFormValidation.js` and `utils/validators.js` don't exist in a fresh scaffold either — the first form in a project creates them once (shared, not per-view), every form after reuses them.

## Skeleton

Canonical form pattern: `validators.js` (`V.*`), `useFormValidation` composable, `.vue`/`.js` form with `novalidate`, `fieldErrors` inline, `*__input--invalid`, submit `loading` state and flash on failure. Never write form markup from memory.

Read [references/skeleton.md](references/skeleton.md) when writing the form markup/script.

## Input types — still required

Setting the correct `type` (`email`, `number`, `date`, `range`, `tel`, ...) is about semantics/mobile keyboard/a11y, not the validation-UI point above — it stays required even with `novalidate`.

## UI component library and confirm/alert dialogs

Canonical rule lives in vulkano-frontend-component § "Installed UI library is not optional" — load it too. Short version: check `package.json`/`PROJECT.md` for whichever UI library this project uses (shadcn-vue, Element Plus, or any other) before any raw `<input>`/`<select>`/etc; if a kit is installed, use it (adding a missing component to the project is expected, not a reason to fall back to HTML5); and never `window.confirm`/`window.alert`/`window.prompt` for confirmations/messages — use the kit's own confirm/message components (or a shared custom modal if no kit is installed).

## Date fields

`type="date"`'s native picker can't be restyled and varies by browser/OS. Check the design system page first per the rule above (the installed library's date-picker if already installed and documented there — e.g. a shadcn-vue `calendar`+`popover` combo or Element Plus's `<el-date-picker>`). If nothing exists yet: `type="date"` is acceptable for low-stakes internal forms and forms not yet carrying the redesign; a view already carrying the redesign should install a date-picker from whichever library this project uses instead — call out the install explicitly, and add it to the design system page.

## Microinteractions (required on every submit)

- `loading` state on submit: disabled button + a loader (`is-loading` is an example class — create it once in `scss/_base.scss` if the project has none), spinner if the action takes noticeable time.
- Success/error flash message (or the UI kit's toast) uses a short transition (~150-250ms), never an instant jump — the `<Transition name="flash">` needs its `.flash-enter-active`/`.flash-leave-active` rules, created once in `scss/_base.scss`.
- Full rules (hover, `cursor: pointer`, shared utilities, GSAP/AOS policy): `references/AGENTS/FRONTEND.md` § Design patterns.

## After writing

- New/changed `validators.js` rule or `useFormValidation` behavior → test at `test/frontend/<entrypoint>/utils/validators.test.js` / `test/frontend/<entrypoint>/composables/useFormValidation.test.js` per vulkano-testing.
- Visually verify in a browser: submit with empty fields, invalid values, and valid values.

## Reference

The Skeleton above (canonical pattern — no pre-existing `frontend/<entrypoint>/views/Login/` to copy from in a fresh scaffold). This skill is the source of truth for form code — `references/AGENTS/FRONTEND.md` § Forms holds only the rules.
