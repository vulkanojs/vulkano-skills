---
name: vulkano-frontend-entrypoint
description: Use when creating a brand-new frontend entrypoint in this Vulkano framework project — a Vue SPA (CMS/admin panel, a custom landing, or any additional app) or a vanilla-JS bundle for a fully server-rendered page — covering the frontend/<name>/ folder scaffold, Vite/nodemon wiring, the backend template/controller/catch-all, and the SEO/Analytics/Accessibility area decision.
---

# Frontend Entrypoint

## Overview

A Vulkano project can have more than one Vue app — public front, CMS/admin, a one-off landing. Each gets its own Vite build entry and its own backend template + controller + catch-all route. This skill is for creating a **new** entrypoint. Not for adding a route inside one that already exists (see vulkano-frontend-router). Not for a component/view's own internals (see vulkano-frontend-component).

**An entrypoint isn't always a Vue SPA.** `vite.entries.mjs`'s `entries` map is just `{ name: 'path/to/file.js' }` — `vite.config.mjs` derives the bundle, alias, and manifest entry from that path alone, with no Vue requirement (the `vue()` plugin only activates for `.vue` files it actually encounters). A fully server-rendered area (Nunjucks/Handlebars, no client-side routing) can get its own entry that's plain JS — no `App.vue`/`routes.js`/Pinia — and still gets Vite's dev-server HMR and a production bundle. See § Vanilla-JS entrypoint below.

## When to use

- Adding a CMS/admin app, a landing page built as its own app, or any second-or-later Vue app to the project
- Adding a compiled/bundled JS entry (with HMR in dev) for a server-rendered page that doesn't need a Vue SPA mount

## Folder placement

`frontend/` is always a container, one subfolder per entrypoint — never flat, even with only 1. The public front is always `frontend/website/` (fixed name); every other entrypoint gets its own `frontend/<name>/` (e.g. `frontend/admin/`). Adding a new entrypoint is just adding another `frontend/<name>/` sibling — no flat-to-container migration to trigger.

```
frontend/
├── website/
│   ├── app.js
│   ├── App.vue
│   ├── routes.js
│   ├── Api.js
│   ├── style.scss
│   ├── components/_index.scss
│   ├── layouts/
│   └── views/_index.scss
└── admin/
    └── ...              # same shape
```

See references/AGENTS/ENTRYPOINTS.md for the full rationale.

## Scaffolding a new entrypoint

Mirror the existing app's shape exactly — same files, per vulkano-frontend-component:

```
frontend/<name>/
  app.js            # Vue entry — mounts App.vue, registers $api
  App.vue           # <router-view></router-view>
  routes.js         # Vue Router routes for this app
  Api.js            # its own fetch wrapper (or share one, project's call)
  style.scss        # this app's own style entry
  components/_index.scss
  layouts/
  views/_index.scss
```

**Alias per entrypoint:** `vite.config.mjs` derives one `resolve.alias` per entrypoint automatically from `vite.entries.mjs` — `@<dir>` for whatever folder a given entry lives under (e.g. `frontend/website/app.js` → `@website`, `frontend/admin/app.js` → `@admin`). Adding the key to `vite.entries.mjs` (see Wiring below) is the only step; no separate alias edit. Use **relative imports** inside each entrypoint (`./style.scss`, `./routes`, `./App.vue`, `./Api`) for its own files; reach for the entrypoint's own alias only when a deep import reads clearer absolute (e.g. `@admin/components/ui/Button.vue` from a nested file). Never reference another entrypoint's alias (`@website` from inside `frontend/admin/`, or vice versa) — entrypoints stay isolated; share code via a `frontend/shared/` folder instead, imported by relative path.

**Router base path — required for any non-root entrypoint:** `createWebHistory()` defaults to base `/`, so Vue Router matches routes against the full URL path with no prefix stripped. An entrypoint mounted under a path prefix (e.g. `/admin`) MUST pass that prefix as the base — `createWebHistory('/admin')` in `app.js` — or every route in `routes.js` silently fails to match once served from the real backend URL (works fine in isolation/dev-root testing, breaks only once the catch-all route actually serves it under its prefix). Only the entrypoint mounted at `/` (the public front) omits the base. This must match the catch-all path registered in `app/config/routes.js` exactly (see Wiring below and vulkano-frontend-router § Multiple entry points).

## Wiring

- **`vite.entries.mjs`** — add a key to the `entries` map (this is the single source of truth `vite.config.mjs` reads both `build.rollupOptions.input` and `resolve.alias` from — no `vite.config.mjs` edit needed):
  ```js
  export const entries = {
    website: 'frontend/website/app.js',
    admin: 'frontend/admin/app.js'
  };
  ```
  Each key is a separate bundle, addressable from a template via `vite({ entry: '<key>', type: '...' })`, and its own `@<dir>` alias is derived automatically (see Alias per entrypoint above).
- **`nodemon.json`** — no change needed. `ignore` already has `frontend/`, which covers every entrypoint under it.
- **Backend template** — `app/views/_shared/templates/<name>.html`, a copy of `default.html` with its own `vite({ entry: '<name>', ... })` calls. Drop the SEO meta block if the area has SEO off (see vulkano-seo).
- **Backend controller** — one per area (e.g. `AdminController.get`), rendering that area's own view (see vulkano-backend-controller):
  ```js
  module.exports = {
    get(req, res) {
      res.render('admin/index.html'); // extends _shared/templates/admin.html
    }
  };
  ```
- **Catch-all route** — in `app/config/routes.js`, scoped to the area's path prefix, registered **before** the generic `/*`:
  ```js
  module.exports = {
    '/': 'HomeController.get',
    '/admin/*': 'AdminController.get', // must come before '/*'
    '/*': 'HomeController.get'
  };
  ```
  See vulkano-frontend-router § Multiple entry points for why the order matters — convention API routes (`/api/*`) register before `config/routes.js` entries, so `/*` never shadows them regardless of where it sits, but a scoped catch-all still needs to precede the generic one or it gets shadowed by it.

## Vanilla-JS entrypoint (server-rendered page, no Vue SPA)

For an area that's fully server-rendered (see vulkano-backend-views-handlebars/-nunjucks) and only needs some compiled/bundled JS — a bit of interactivity, a third-party widget init, progressive enhancement — not a client-routed app:

```
frontend/<name>/
  app.js            # plain JS entry — no App.vue, no routes.js, no Api.js/Pinia unless this page actually calls the API
  style.scss        # this entry's own style entry, imported from app.js (or injected separately, project's call)
```

No `views/`/`components/`/`layouts/` folders unless the page's JS is genuinely split into multiple reusable pieces — most vanilla entries are one file. Skip the Vue-specific steps below: no `createWebHistory(base)` (there's no Vue Router), no `<div id="app">` mount point in the backend template, no frontend catch-all (the page is server-rendered per normal controller/view routing — see vulkano-backend-views — so a hard refresh always hits the real controller, not a client router).

Wiring is otherwise identical — same `vite.entries.mjs` key, same `@<name>` alias, same `vite({ entry: '<name>', type: '...' })` calls in the backend template (see Wiring below), same HMR in dev. The only difference is what's on the other end of that bundle: a `createApp(App).mount('#app')` call for a Vue entry, vs. whatever plain DOM code this page needs for a vanilla one.

## SEO / Analytics / Accessibility per area

Every new entrypoint is a new "area" per PROJECT.md § Project requirements. Before considering the entrypoint done:

1. Ask the user which kind of area this is (landing / landing+form / website / blog, embeddable widget, or CMS/admin panel) if not already stated.
2. Map it to SEO/Analytics/Accessibility on/off per the existing rule in PROJECT.md.
3. Add a row for it to the table in PROJECT.md § Project requirements, and show the user the result.

## After writing

- Confirm the new key was added to `vite.entries.mjs` — its `@<name>` alias and `rollupOptions.input`/`environments` entry in `vite.config.mjs` both derive from it automatically, nothing else to edit.
- Run `vp build` and confirm the new entry appears in `public/.vite/manifest.json` — each entry builds as its own isolated Rolldown pass (Vite's Environment API, see `vite.config.mjs`) so it never shares a chunk with another entry, then all entries' manifests merge into that one file.
- Vue entrypoint: hit the new area's path in a browser; confirm a hard refresh doesn't 404 (catch-all working, not just in-app navigation) AND the view actually renders — a 200 with a blank page is the tell for a missing/wrong `createWebHistory(base)` (see Router base path above), not a catch-all problem.
- Vanilla-JS entrypoint: confirm the bundle actually loads on the rendered page (Network tab) and, in dev, that editing `app.js` HMR-updates without a full page reload — no catch-all/router check needed, it's not a client-routed app.
- Run `vp check` and `vp test`.

## Reference

references/AGENTS/ENTRYPOINTS.md, vulkano-frontend-router § Multiple entry points / Backend catch-all, vulkano-backend-controller, vulkano-seo, PROJECT.md § Project requirements.
