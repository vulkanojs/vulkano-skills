## design system page

Every entrypoint keeps a dev-only design system view — `frontend/<entrypoint>/views/DesignSystem/Index.vue`, route `/design-system` — listing every reusable component with a live usage example and its props/slots. This page is the reuse source of truth: check it before writing a new component or raw form field, so nothing gets reinvented that already exists.

Two kinds of components live behind it, both get an entry:

- **Custom components** (Datepicker, Textarea, Select, Modal, Button, etc.) — code lives in `components/<Name>/`.
- **Vendored UI-library components** (shadcn-vue, Element Plus) — code lives in `components/ui/`.

`components/` (and `components/ui/`) hold the code; `views/DesignSystem/Index.vue` holds nothing but examples — it never re-implements a component, only imports and demonstrates it.

- Ships in the template as the entrypoint's design system page (tokens, typography, spacing, base components, grid); every reusable component adds its own section to it.
- The route is registered only when `import.meta.env.DEV` (see the entrypoint's `routes.js`) — production builds don't include it.
- One section per component: name, short description, a rendered live instance, and the exact `<template>` snippet to copy-paste (props included) — mirror that snippet verbatim when reusing, don't improvise a variant.
- Keep entries flat, one per component — no nested taxonomy needed for this.
