---
name: vulkano-frontend-router
description: Use when adding a Vue Router route, an auth/login guard, or a route-based redirect inside an existing entrypoint in this Vulkano framework project's frontend/ — route↔view naming, resource/action file naming, the SPA catch-all on the backend, and fetching the current user (never caching it client-side). For creating a brand-new entrypoint (CMS/admin, a custom landing, etc.), see vulkano-frontend-entrypoint instead.
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

Every non-root entrypoint needs that scope — see § Router base path below for why omitting it breaks routing once served for real (not just in isolated dev testing).

Route path and view folder mirror each other (kebab-case URL → PascalCase folder) — see vulkano-frontend-component for the full naming convention.

**Kebab-case means hyphen at every word boundary, not just lowercasing.** Split the PascalCase folder name at each word: `VatTypes` → `/vat-types`, not `/vattypes`. Same for multi-word resource/action segments (`ProductCategory` → `/product-category`). Applies to every route segment, including nested sections (`/config/vat-types`, not `/config/vattypes`).

Since `@vulkano/core` 1.25.0 the backend's convention-based API routing applies the same hyphenation (`toKebabCase()` in `controllers/controllers.js`), so `VatTypesController` auto-resolves to `/api/vat-types/…`, matching the frontend path one-to-one — see vulkano-backend-controller. (Before 1.25.0 the backend only lowercased the controller name with no hyphen insertion, so `/api/vat-types/` 404'd under the convention — that gap is closed as of this version.)

### Resource + action routes — `domain.com/<resource>/<action>`

When a route is a resource with multiple actions (`/product/list`, `/product/edit`), don't nest a new folder+`Index.vue` per action — that's a needless single-file subfolder per action. Instead the resource is the folder, and each action is its own named file inside it:

```
views/
  Product/
    List.vue / List.js / _list.scss
    Form.vue / Form.js / _form.scss
```

```js
{ path: '/product/list', component: () => import('./views/Product/List.vue') },
{ path: '/product/create', component: () => import('./views/Product/Form.vue') },
{ path: '/product/edit/:id', component: () => import('./views/Product/Form.vue') }
```

This only replaces the leaf filename — it does not apply to plain single-segment or nested-section routes (`/users`, `/system/users`), which keep the existing `Index.vue`/`Index.js` leaf convention (see vulkano-frontend-component). Use resource+action naming specifically when a folder holds more than one action for the same resource — it's what makes `domain.com/product/edit` map straight to `views/Product/Form.vue` without grepping.

### Create + Edit → one `Form.vue`, not `Create.vue`/`Edit.vue`

Don't split create and edit into two separate views — they're the same form, branching only on whether `:id` is present in the route. One `Form.vue` (routed from both `/product/create` and `/product/edit/:id`) avoids duplicating the whole form markup/validation across two files:

```js
// Form.js
import { ref, onMounted, getCurrentInstance } from 'vue';
import { useRoute, useRouter } from 'vue-router';

export default {
  setup() {
    const { $api } = getCurrentInstance().proxy || {};
    const route = useRoute();
    const router = useRouter();
    const id = route.params.id; // undefined on /product/create
    const form = ref({ name: '', price: 0 });

    onMounted(async () => {
      if (id) form.value = await $api.get(`/product/${id}`); // getData for edit
    });

    async function submit() {
      if (id) {
        await $api.put(`/product/${id}`, form.value);
      } else {
        await $api.post('/product', form.value);
      }
      router.push('/product/list');
    }

    return { form, submit };
  }
};
```

Same rule applies to the backend controller (`PUT`/`POST` split on `:id`, not two controllers) — see vulkano-backend-controller.

## Backend catch-all — required for hard refresh

`app/config/routes.js` must keep a catch-all as its **last** entry so a hard refresh or direct URL hit on any client route returns the SPA's `index.html` instead of a 404:

```js
module.exports = {
  '/': 'HomeController.get',
  '/*': 'HomeController.get' // must stay last
};
```

Without it, every non-`/` client route 404s on hard refresh/direct URL while still working via in-app navigation — that split symptom (`<router-link>` works, refresh 404s) is the tell this is missing. Safe to keep last: convention API routes (`app/controllers/api/*`) register before `config/routes.js` entries, so `/*` never shadows an API route.

**Only add a `/*` (or scoped `/<area>/*`) catch-all for an area that actually mounts a Vue app** — check that its backend template has `<div id="app">` + `{{ vite({ entry: '<name>' }) }}` (e.g. `app/views/_shared/templates/default.html`). A fully server-rendered area (extending `_shared/templates/static.html` or similar, no Vue mount point) must **not** get a blanket `/*`: `@vulkano/core` already returns a real `404` via its built-in handler (`app/views/_shared/errors/404.html`) for any unmatched route in that area, and a catch-all there would intercept it and soft-200 every invalid URL into the homepage instead — wrong for SEO and for users. When adding the catch-all for a mounted area, always add both halves together, never one without the other:

1. **Backend**: the scoped (or generic) catch-all above.
2. **Frontend**: a Vue Router catch-all in that entrypoint's `routes.js` — `{ path: '/:pathMatch(.*)*', component: NotFound }` — rendering a `views/NotFound/Index.vue`, so an invalid path inside that area shows a real 404 UI instead of silently re-rendering the home route.

### Multiple entry points (e.g. a separate `/admin` area)

Creating a brand-new entrypoint (a CMS/admin app, a custom landing, or any other split app separate from the public front) — the `frontend/<name>/` folder scaffold, Vite/nodemon wiring, and the backend template/controller/catch-all scaffold — is covered by vulkano-frontend-entrypoint. Use it whenever the task is adding the entrypoint itself, not just a route inside one that already exists.

Once that entrypoint exists, each one still needs its **own** backend catch-all, scoped to its path prefix, registered **before** the generic `/*` so the more specific pattern isn't shadowed by it:

```js
module.exports = {
  '/': 'HomeController.get',
  '/admin/*': 'AdminController.get', // scoped catch-all for the admin entry — must come before '/*'
  '/*': 'HomeController.get' // public front catch-all — must stay last
};
```

`AdminController.get` renders the admin entry's own `index.html`/template (its own `vite({ entry: 'admin' })` bundle, not the public front's `app` entry) — the two SPAs don't share a bundle just because they share the Express process.

The scoped entrypoint's own `app.js` must pass the same prefix as the Vue Router base (`createWebHistory('/admin')`) — see vulkano-frontend-entrypoint § Router base path. Mismatch between the two is a common miss: the backend route serves the app fine, but every client-side route inside it fails to match because the router is still expecting `/`.

## Auth guard — never cache user client-side

No `localStorage`/`sessionStorage` for the token or the user object — both are XSS-exposed. The JWT lives in an `httpOnly` cookie (set by the backend on login) — the frontend never reads or stores it directly. See vulkano-backend-auth for the cookie/`AuthController` convention on the backend side.

Re-fetch the current user on every route change instead of caching it in a store across navigations:

```js
// frontend/<entrypoint>/routes.js (or a separate router/guards.js imported here)
router.beforeEach(async (to) => {
  const isAuthRoute = to.path === '/login';
  let user = null;
  try {
    user = await Api.get('/auth/current'); // 401 → rejects
  } catch (_err) {
    user = null;
  }

  if (!user && !isAuthRoute) return '/login';
  if (user && isAuthRoute) return '/';
  return true;
});
```

`GET /api/auth/current` is the backend's session-check endpoint — see vulkano-backend-auth for the `AuthController` convention it expects (`login`/`logout`/`current` method keys).

## After writing

- Confirm the backend `/*` catch-all still exists if this is the first route added to a fresh project.
- Test hard-refresh on the new route, not just in-app navigation.
- Run `vp check` and `vp test`.

## Reference

`references/AGENTS/ROUTING.md` (backend catch-all, cross-cutting), `references/AGENTS/FRONTEND.md` § Security, vulkano-backend-auth (backend session-check convention).
