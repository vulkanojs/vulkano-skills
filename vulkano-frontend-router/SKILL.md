---
name: vulkano-frontend-router
description: "Use when adding a Vue Router route, an auth/login guard, or a route-based redirect inside an existing entrypoint in this Vulkano framework project's frontend/ — route↔view naming, resource/action file naming, the SPA catch-all on the backend, and fetching the current user (never cached client-side). New entrypoint: vulkano-frontend-entrypoint."
---

# Frontend Router

## Overview

`frontend/<entrypoint>/routes.js` is a hand-written route array (Vue Router, HTML5 history mode) — one per entrypoint subfolder, always (`frontend/website/routes.js`, `frontend/admin/routes.js`, ...), each with its own routes. No auto-discovery of `views/`. Auth state is never cached client-side — every route change re-fetches the current user from the backend.

## When to use

- Adding a new route/view
- Adding a login/auth guard
- Any redirect-based access control (redirect to `/login` if unauthenticated, redirect away from `/login` if already authenticated)

Not for the view's own file layout — see vulkano-frontend-component. Not for the backend `AuthController`/JWT cookie setup — see vulkano-backend-auth. Not for creating a brand-new entrypoint — see vulkano-frontend-entrypoint.

## Adding a route

```js
// frontend/<entrypoint>/routes.js
import { createRouter } from 'vue-router';

import Layout from './layouts/Layout.vue';
import Homepage from './views/Home/Index.vue';
import Users from './views/System/Users/Index.vue';

const routes = [
  {
    path: '/',
    component: Layout,
    children: [
      { path: '', component: Homepage },
      { path: '/system/users', component: Users }
    ]
  }
];

export default (history) => createRouter({ history, routes });
```

`routes.js` takes the `history` object rather than creating it — the actual `createWebHistory(...)` call, and its base-path scope, lives in `app.js`:

```js
// frontend/website/app.js — public front, mounted at '/', no base
const router = createRouter(createWebHistory());
```

```js
// frontend/admin/app.js — mounted at '/admin', base MUST match
const router = createRouter(createWebHistory('/admin'));
```

Every non-root entrypoint needs that scope — see vulkano-frontend-entrypoint § Router base path for why omitting it breaks routing once served for real (not just in isolated dev testing).

Route path and view folder mirror each other (kebab-case URL → PascalCase folder) — see vulkano-frontend-component for the full naming convention.

**Kebab-case means hyphen at every word boundary, not just lowercasing.** Split the PascalCase folder name at each word: `VatTypes` → `/vat-types`, not `/vattypes`. Same for multi-word resource/action segments (`ProductCategory` → `/product-category`). Applies to every route segment, including nested sections (`/config/vat-types`, not `/config/vattypes`).

The backend hyphenates the same way (`VatTypesController` → `/api/vat-types/…`), so frontend and API paths match one-to-one — see vulkano-backend-controller.

### Resource + action routes — `domain.com/<resource>/<action>`

When a route is a resource with multiple actions (`/product/list`, `/product/edit`), don't nest a new folder+`Index.vue` per action — that's a needless single-file subfolder per action. Instead the resource is the folder, and each action is its own named file inside it:

```
views/
  Product/
    _index.scss          # the folder's entry — imports the partials below
    List.vue / List.js / _list.scss
    Form.vue / Form.js / _form.scss
```

`_index.scss` stays the one file the parent aggregator imports; `_list.scss`/`_form.scss` are partials it pulls in (`@import './list';`) so no single stylesheet grows long.

```js
{ path: '/product/list', component: () => import('./views/Product/List.vue') },
{ path: '/product/create', component: () => import('./views/Product/Form.vue') },
{ path: '/product/edit/:id', component: () => import('./views/Product/Form.vue') }
```

This only replaces the leaf filename — it does not apply to plain single-segment or nested-section routes (`/users`, `/system/users`), which keep the existing `Index.vue`/`Index.js` leaf convention (see vulkano-frontend-component). Use resource+action naming specifically when a folder holds more than one action for the same resource — it's what makes `domain.com/product/edit` map straight to `views/Product/Form.vue` without grepping.

### Create + Edit → one `Form.vue`, not `Create.vue`/`Edit.vue`

One `Form.vue` routed from both `/product/create` and `/product/edit/:id`, branching only on whether `:id` is present (`$api.put` vs `$api.post`).

Read [references/create-edit-form.md](references/create-edit-form.md) when writing a create/edit view.

## NotFound route + backend catch-all

A route inside a Vue entrypoint only survives a hard refresh if the backend has that entrypoint's catch-all — rule in `references/AGENTS/BACKEND.md` § Routes, wiring in vulkano-frontend-entrypoint § Wiring. The frontend half: the **last child of the `Layout` route** in the same `routes.js` — a child, so the 404 renders inside the page template:

```js
import NotFound from './views/NotFound/Index.vue';

{
  path: '/',
  component: Layout,
  children: [
    // ...every other route
    { path: ':pathMatch(.*)*', component: NotFound } // last
  ]
}
```

Missing backend half → `<router-link>` works but a hard refresh 404s.

## Auth guard — never cache user client-side

No `localStorage`/`sessionStorage` for the token or user; JWT lives in an `httpOnly` cookie. Mark protected routes `meta: { auth: true }` and re-fetch the current user (`GET /api/auth/current`) on each navigation in `router.beforeEach`. The guard is UX only — the backend enforces auth.

Read [references/auth-guard.md](references/auth-guard.md) when writing the guard.

## After writing

- First non-root route in an entrypoint → its catch-all must be active in `app/config/routes.js` (`references/AGENTS/BACKEND.md` § Routes).
- Test hard-refresh on the new route, not just in-app navigation.

## Reference

`references/AGENTS/BACKEND.md` § Routes (backend catch-all), `references/AGENTS/FRONTEND.md` § Routes, vulkano-backend-auth (backend session-check convention).
