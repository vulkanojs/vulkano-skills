---
name: vulkano-frontend-form
description: MANDATORY — load BEFORE writing or editing ANY `<form>` in this Vulkano project's frontend/, no exceptions, even a "quick" one-field form. Trigger words — form, `<form>`, input, field, validation, required field, contact form, login form, signup, submit button, CRUD create/edit view. Covers required-field asterisks, JS-only rules-based validation via the shared useFormValidator composable, fieldErrors pattern, input types, and date-picker choice. Do not write form markup/logic from memory of a past form — conventions in this skill may have moved.
---

# Frontend Form

## Overview

Forms never rely on native browser validation UI (`required`/`:invalid` styling, error bubbles) — it can't be styled consistently and breaks the design system. Validation is always hand-rolled in JS: a rules object per field, checked by the shared `useFormValidator` composable, error state exposed as `fieldErrors`, message rendered inline.

`useFormValidator` is a project-owned, dependency-free composable — not a third-party validation library. If a validation library is added to the project later, swap the composable's internals only; every form's `formRules`/`validate(...)` usage stays unchanged.

## When to use

Any `<form>` add/edit in `frontend/` — contact forms, login, CRUD create/edit views.

Not for component file layout — see vulkano-frontend-component. Not for a11y attributes on the form (labels, `aria-invalid`) — see vulkano-frontend-a11y, apply both together.

## Required-field pattern

- Validation runs through `const { fieldErrors, validate } = useFormValidator(form, formRules)` (see Skeleton) — never native `required`/`:invalid` UI.
- `formRules` is a plain object: `{ email: [V.required('...'), V.email('...')] }` — each entry an array of validator functions from `frontend/<entrypoint>/utils/validators.js` (`V.required(message)`, `V.email(message)`, ...), checked in order, first failing rule wins.
- `fieldErrors` is the reactive object returned by `useFormValidator` — never re-declared locally.
- Every required field's label gets a red asterisk: reuse a shared `.field-required` (or equivalent BEM element) styled with the project's danger/error color token — never hardcode a hex/named red per view. Token name and scale (`--color-danger`, `--color-danger-500`, `--color-error`, whatever this project's own theme uses) is a project decision, not a framework-fixed name — check for an existing one first. **Neither the class nor a danger/error token exists in a fresh scaffold** (checked: no `frontend/**/*.scss` defines one) — the first form in a project defines both once, in a shared partial (e.g. `frontend/<entrypoint>/scss/_tokens.scss`, imported from `style.scss`), and every form after that reuses them.
- Error message rendered inline below the input: `<span class="*__field-error">{{ fieldErrors.email }}</span>`.
- Invalid input gets a `*__input--invalid` class for the red border.
- Required inputs keep the native `required` attribute even though `novalidate` suppresses its browser UI — not redundant with `V.required(...)` in `formRules`: it stays for the accessibility tree, gives a quick visual signal in devtools without opening `formRules`, and a mismatch between the two (required in markup but not in rules, or vice versa) is a bug worth catching. See vulkano-frontend-a11y § Forms.
- No `frontend/<entrypoint>/views/Login/` exists in a fresh scaffold — it's not a file to go open and copy. Follow the `useFormValidator` + `formRules` + `fieldErrors` + `<span class="*__field-error">` + `*__input--invalid` shape from the Skeleton below instead; once a project's first login/form view exists, treat _that_ as the local reference for the next one.
- `composables/useFormValidator.js` and `utils/validators.js` don't exist in a fresh scaffold either — the first form in a project creates them once (shared, not per-view), every form after reuses them.

## Skeleton

```js
// frontend/<entrypoint>/utils/validators.js
export const V = {
  required: (message) => (value) => (value === '' || value == null ? message : null),
  email: (message) => (value) => (/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? null : message)
};
```

```js
// frontend/<entrypoint>/composables/useFormValidator.js
import { ref } from 'vue';

export function useFormValidator(model, rules) {
  const fieldErrors = ref({});

  async function validate(callback) {
    const errors = {};

    for (const field in rules) {
      for (const rule of rules[field]) {
        const message = rule(model[field]);
        if (message) {
          errors[field] = message;
          break;
        }
      }
    }

    fieldErrors.value = errors;
    const isValid = Object.keys(errors).length === 0;
    callback(isValid);
    return isValid;
  }

  return { fieldErrors, validate };
}
```

```js
// Index.js
import { reactive, ref, getCurrentInstance } from 'vue';
import { V } from '@website/utils/validators.js';
import { useFormValidator } from '@website/composables/useFormValidator.js';

export default {
  setup() {
    const { $api } = getCurrentInstance().proxy || {};
    const form = reactive({ email: '', password: '' });
    const isSubmitting = ref(false);

    const formRules = {
      email: [V.required('Email is required'), V.email('Enter a valid email address')],
      password: [V.required('Password is required')]
    };

    const { fieldErrors, validate } = useFormValidator(form, formRules);

    async function save() {
      isSubmitting.value = true;
      try {
        await $api.post('/auth/login', form);
      } finally {
        isSubmitting.value = false;
      }
    }

    async function submit() {
      await validate((isValid) => {
        if (isValid) {
          save();
          return true;
        }
        return false;
      });
    }

    return { form, fieldErrors, isSubmitting, submit };
  }
};
```

```html
<form novalidate @submit.prevent>
  <label class="login__label">
    Email <span class="field-required">*</span>
    <input
      id="email"
      type="email"
      required
      v-model="form.email"
      :class="{ 'login__input--invalid': fieldErrors.email }"
      :aria-invalid="!!fieldErrors.email"
      :aria-describedby="fieldErrors.email ? 'email-error' : null"
    />
  </label>
  <span v-if="fieldErrors.email" id="email-error" class="login__field-error"
    >{{ fieldErrors.email }}</span
  >

  <button
    type="submit"
    :disabled="isSubmitting"
    :class="{ 'is-loading': isSubmitting }"
    @click="submit"
  >
    Submit
  </button>
</form>
```

## Input types — still required

Setting the correct `type` (`email`, `number`, `date`, `range`, `tel`, ...) is about semantics/mobile keyboard/a11y, not the validation-UI point above — it stays required even with `novalidate`.

## UI component library and confirm/alert dialogs

Canonical rule lives in vulkano-frontend-component § "Installed UI library is not optional" — load it too. Short version: check `package.json`/`PROJECT.md` for whichever UI library this project uses (shadcn-vue, Element Plus, or any other) before any raw `<input>`/`<select>`/etc; if a kit is installed, use it (adding a missing component to the project is expected, not a reason to fall back to HTML5); and never `window.confirm`/`window.alert`/`window.prompt` for confirmations/messages — use the kit's own confirm/message components (or a shared custom modal if no kit is installed).

## Date fields

`type="date"`'s native picker can't be restyled and varies by browser/OS. Check the UI reference page first per the rule above (the installed library's date-picker if already installed and documented there — e.g. a shadcn-vue `calendar`+`popover` combo or Element Plus's `<el-date-picker>`). If nothing exists yet: `type="date"` is acceptable for low-stakes internal forms and forms not yet carrying the redesign; a view already carrying the redesign should install a date-picker from whichever library this project uses instead — call out the install explicitly, and add it to the UI reference page.

## Microinteractions (required on every submit)

- `loading` state on submit: disabled/`is-loading` button, spinner if the action takes noticeable time.
- Success/error toast or inline message uses a short transition (~150-250ms), never an instant jump.

## After writing

- Track both outcomes (`{section}_success` / `{section}_error`) per vulkano-frontend-analytics, unless the user opted out for this area.
- Confirm a11y requirements (labels, `aria-invalid`, `aria-describedby`) per vulkano-frontend-a11y.
- New/changed `validators.js` rule or `useFormValidator` behavior → test at `test/frontend/<entrypoint>/utils/validators.test.js` / `.../composables/useFormValidator.test.js` per vulkano-testing.
- Visually verify in a browser: submit with empty fields, invalid values, and valid values.
- Run `vp check` and `vp test`.

## Reference

The Skeleton above (canonical pattern — no pre-existing `frontend/<entrypoint>/views/Login/` to copy from in a fresh scaffold), references/AGENTS/ACCESSIBILITY.md § Forms. AGENTS.md § Form fields only points here now — this skill is the source of truth, not a summary of it.
