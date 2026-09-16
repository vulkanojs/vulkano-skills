---
name: vulkano-frontend-component
description: Use when creating, editing, or reviewing a Vue component or view in this Vulkano framework project's frontend/ folder — .vue/.js/.scss file splitting, Composition API, views/ vs components/ placement, route↔view naming, Pinia store per concern, BEM styling, CSS Grid layout, and which UI kit (shadcn-vue/Element Plus) owns inputs, dialogs, and confirm/alert. MANDATORY before writing any delete/confirm action or `window.confirm`/`window.alert`/`window.prompt` call, even outside a form.
---

# Frontend Component

## Overview

`frontend/<entrypoint>?/` (flat `frontend/` with 1 entrypoint, one subfolder per app once 2+ — see vulkano-frontend-entrypoint) is a Vue 3 SPA (Composition API) bundled by Vite. Every component/view splits template, logic, and styles into sibling files — never a single-file `<script setup>` block with inline everything.

## When to use

- New view/component needed under `frontend/<entrypoint>?/views/` or `frontend/<entrypoint>?/components/`
- Adding/editing a Pinia store
- Wiring a new route
- Styling a component/view (BEM, grid)

Not for form validation UI — see vulkano-frontend-form. Not for route/auth-guard wiring — see vulkano-frontend-router. Not for analytics wiring — see vulkano-frontend-analytics. Not for accessibility — see vulkano-frontend-a11y. Not for backend — see vulkano-backend-* skills.

## Before implementing

- Composition API only (`setup()`, `ref`/`reactive`, composables) — never add `data()`/`methods`/`created()` options blocks.
- Decide `views/` (top-level route-driven state) vs `components/` (reusable, imported by views/other components).
- Check the UI reference page (see below) first — it's the reuse source of truth, faster and more reliable than grepping `frontend/<entrypoint>?/components/`.
- Check for an existing similar view/component — mirror its file layout instead of inventing a new one.
- Form fields especially: check if a reusable input component already exists project-wide (Datepicker, Textarea, Select, etc.) before adding a raw `<input>`/`<textarea>`. If site/CMS already uses one (e.g. Datepicker instead of `<input type="date">`), reuse it — copy its exact component/placement pattern, don't invent a new one.

## File & naming

**Component** (`frontend/<entrypoint>?/components/<Name>/`):

```
components/
  _index.scss              # aggregator — every component adds @import here
  MyComponent/
    MyComponent.vue         # template only
    MyComponent.js          # logic, imported via <script src="./MyComponent.js">
    _index.scss             # styles (BEM)
```

**View** (`frontend/<entrypoint>?/views/<Path>/`) — leaf file always `Index.vue`/`Index.js`, folder name identifies the view:

```
views/
  _index.scss
  Users/Index.vue            # /users
  System/Users/Index.vue     # /system/users (nested route → nested folder)
```

A module folder (`System/`) gets its own `_index.scss` aggregator importing its children's, same pattern one level deeper.

Route path and view folder always mirror each other (kebab-case URL → PascalCase folder) — no code generates this, keep it by hand so a route is locatable without grepping.

Exception — resource+action routes (`/product/list`, `/product/create`, `/product/edit/:id`): the resource is the folder, each action is its own named file (`Product/List.vue`, `Product/Form.vue`) instead of an `Index.vue` per action-folder. Create and edit share one `Form.vue` (branch on `:id` presence: `GET` to prefill + `PUT` when editing, `POST` when creating) rather than separate `Create.vue`/`Edit.vue` files. See vulkano-frontend-router § Resource + action routes.

## Component/view skeleton

```js
// MyComponent.js
import { ref, onMounted, getCurrentInstance, toRef } from 'vue';

export default {
  setup(props) {
    const { $api } = getCurrentInstance().proxy || {}; // never `import Api` directly
    const sku = toRef(props, 'sku');
    const products = ref([]);

    onMounted(async () => {
      products.value = await $api.get('/product');
    });

    return { products, sku };
  }
};
```

```html
<!-- MyComponent.vue -->
<script src="./MyComponent.js"></script>
<template>
  <div class="my-component">...</div>
</template>
```

`$api` is a global property (`app.config.globalProperties.$api`), not an importable module — always pull it off `getCurrentInstance().proxy`.

## Images / static assets

Never `src="@website/..."`/`src="@admin/..."`, a relative `import`, or any other Vite-bundled reference for an image, font, or downloadable file — those live in `public/` and are served as-is by Express, not bundled. Reference them by absolute path from the app root:

```html
<img src="/img/logo.png" alt="Vulkano logo" />
```

`/img/`, `/fonts/`, `/files/` — never `public/` in the path, never through an entrypoint alias (`@website`, `@admin`). See AGENTS.md § Frontend assets for namespacing and `.webp` optimization.

## UI reference page

Every entrypoint keeps a dev-only UI reference view — `frontend/<entrypoint>?/views/UI/Index.vue`, route `/ui` — listing every reusable component with a live usage example and its props/slots. This page is the reuse source of truth: check it before writing a new component or raw form field, so nothing gets reinvented that already exists.

Two kinds of components live behind it, both get an entry:

- **Custom components** (Datepicker, Textarea, Select, Modal, Button, etc.) — code lives in `components/<Name>/`.
- **Vendored UI-library components** (shadcn-vue, Element Plus — see root CLAUDE.md § UI components) — code lives in `components/ui/`.

`components/` (and `components/ui/`) hold the code; `views/UI/Index.vue` holds nothing but examples — it never re-implements a component, only imports and demonstrates it.

- Not built yet in a fresh scaffold — the first reusable component in an entrypoint creates the page; every reusable component after adds itself to it.

## Installed UI library is not optional — check `package.json` before native HTML

**Check for `components.json` (repo root) or `package.json`'s `element-plus` first, not just the reference page above.** `components.json` is shadcn-vue's own canonical marker — always created by `shadcn-vue init` regardless of which underlying primitives library it pulled in that version (`reka-ui` today, `radix-vue` in older setups, could rename again) — don't grep for a specific peer-dep name, it drifts. Either marker present means that library is this project's chosen UI kit — every input, button, dialog, and confirm/alert MUST go through it, not raw HTML5 or a native browser dialog. An empty/missing entry in `views/UI/Index.vue` is not license to fall back — it means the component hasn't been added to this project yet:

1. Check `views/UI/Index.vue` for an existing usage example — copy it verbatim if found.
2. If the kit is installed but this component isn't in the reference page yet: add it now (`pnpm dlx shadcn-vue add <component>`, or use the already-imported Element Plus tag), then add its usage example to the reference page. Expected first-use cost, not an edge case to route around.
3. Only fall back to plain HTML5/native browser UI when neither `components.json` nor `element-plus` is present — say so explicitly in the response (e.g. "No hay shadcn-vue/Element Plus instalado, usé `<input type=\"date\">` nativo"). Never fall back silently, and never fall back just because the reference page happened to be empty.

**`window.confirm`/`window.alert`/`window.prompt` are always off-limits** for user-facing confirmation or messaging (delete confirmations, save feedback, etc.) — native dialogs block the main thread and can't be styled, same reason native form-validation UI is banned (see vulkano-frontend-form):

- shadcn-vue installed → `AlertDialog` (`pnpm dlx shadcn-vue add alert-dialog` if not yet added) for confirmations, `Sonner`/`Toast` for messages.
- Element Plus installed → `ElMessageBox.confirm(...)` for confirmations, `ElMessage`/`ElNotification` for messages.
- Neither installed → build one minimal reusable confirm-modal/toast component (shared, not per-view) instead of reaching for `window.confirm`.
- Guard the route (dev/local only, or behind admin auth) — it's a build tool, never a public/indexed page. No SEO, no analytics.
- One section per component: name, short description, a rendered live instance, and the exact `<template>` snippet to copy-paste (props included) — mirror that snippet verbatim when reusing, don't improvise a variant.
- Keep entries flat, one per component — no nested taxonomy needed for this.

## Routing

See vulkano-frontend-router for route wiring and the backend catch-all requirement.

## State — Pinia, one store per concern

`store/use<Entity>Store.js`, setup-style (not options-style), no barrel file:

```js
export const useEventStore = defineStore('event', () => {
  const { $api } = getCurrentInstance().proxy || {};
  const current = ref(null);
  async function fetch(id) {
    current.value = await $api.get(`/event/${id}`);
  }
  return { current, fetch };
});
```

Exception: app-shell-wide singleton state (loading spinner, socket status, sidebar) goes in one `useAppStore` — everything else stays split per entity.

## Layout — CSS Grid only

`display: grid` everywhere in `_index.scss`, never Flexbox. Use the responsive grid system (`frontend/<entrypoint>?/scss/_grid.scss`): `.row` (`grid-template-columns: repeat(12, 1fr)`) + `.column.small-N.medium-N.large-N`.

## Styling — BEM

```scss
.my-component {
  display: grid;
  &__container {
    display: grid;
  }
  &--opened {
    /* state modifier */
  }
}
```

Block name = component/view folder in kebab-case. No reaching into a child block's internals from a parent stylesheet.

## After writing

- Add the component's `@import './X/_index.scss';` line to the parent `_index.scss` aggregator.
- New reusable component → add its section to the UI reference page (`frontend/<entrypoint>?/views/UI/Index.vue`).
- New store → test at `test/store/use<Entity>Store.test.js` (`createPinia()` + `setActivePinia()` in `beforeEach`, mock `$api` at the store boundary).
- Check analytics (vulkano-frontend-analytics) and accessibility (vulkano-frontend-a11y) requirements for the area before considering done.
- Visually verify in a browser per AGENTS.md § Visual verification.
- Run `vp check` and `vp test`.

## Reference

`reference/FRONTEND.md` (full detail), `reference/ARCHITECTURE.md`.
