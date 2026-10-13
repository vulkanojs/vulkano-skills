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
    const isSubmitting = ref(false);
    const error = ref('');

    async function getProduct(id) {
      try {
        form.value = await $api.get(`/product/${id}`); // prefill for edit
      } catch (err) {
        error.value = err.message;
      }
    }

    onMounted(async () => {
      if (!id) return;
      await getProduct(id);
    });

    async function submit() {
      // validate first — useFormValidation, see vulkano-frontend-form (skipped here)
      isSubmitting.value = true;
      error.value = '';
      try {
        if (id) {
          await $api.put(`/product/${id}`, form.value);
        } else {
          await $api.post('/product', form.value);
        }
        router.push('/product/list');
      } catch (err) {
        error.value = err.message; // backend error.detail
      } finally {
        isSubmitting.value = false;
      }
    }

    return { form, isSubmitting, error, submit };
  }
};
```

Only the `:id` branching is router-specific — fields, validation, flash and loading follow vulkano-frontend-form. Same rule applies to the backend controller (`PUT`/`POST` split on `:id`, not two controllers) — see vulkano-backend-controller.

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

No `localStorage`/`sessionStorage` for the token or the user object — both are XSS-exposed. The JWT lives in an `httpOnly` cookie (set by the backend on login) — the frontend never reads or stores it directly. See vulkano-backend-auth for the cookie/`AuthController` convention on the backend side.

Mark protected routes with `meta: { auth: true }` and re-fetch the current user on each navigation to one — never cache it in a store across navigations. `to.meta` merges parent metas, so a whole area (e.g. admin) is protected by putting `auth: true` on its `Layout` route once.

```js
// frontend/<entrypoint>/routes.js
import { createRouter } from 'vue-router';
import Api from './Api'; // no component instance in a guard — import, not $api

import Layout from './layouts/Layout.vue';
import Login from './views/Login/Index.vue';
import Dashboard from './views/Dashboard/Index.vue';

const routes = [
  { path: '/login', component: Login },
  {
    path: '/',
    component: Layout,
    meta: { auth: true }, // every child requires a session
    children: [{ path: '', component: Dashboard }]
  }
];

export default (history) => {
  const router = createRouter({ history, routes });

  router.beforeEach(async (to) => {
    const isLogin = to.path.startsWith('/login'); // also /login/2fa, /login/forgot…
    if (!to.meta.auth && !isLogin) {
      return true; // public route
    }

    const user = await Api.get('/auth/current').catch(() => null); // 401 → null

    if (!user && !isLogin) {
      return { path: '/login', query: { redirect: to.fullPath } };
    }
    if (user && isLogin) {
      return to.query.redirect || '/';
    }
    return true;
  });

  return router;
};
```

Entrypoint entirely behind login (e.g. admin) — same guard without the `meta` check: every route except `/login*` requires a session, so nothing needs marking:

```js
router.beforeEach(async (to) => {
  const isLogin = to.path.startsWith('/login');
  const user = await Api.get('/auth/current').catch(() => null);

  if (!user && !isLogin) {
    return '/login';
  }
  if (user && isLogin) {
    return '/';
  }
  return true;
});
```

Use `meta.auth` when the entrypoint mixes public and private routes (e.g. `website` with a members area); use the whole-area guard when nothing is public.

The guard is UX only — the real boundary is the backend: every `/api` route a protected view calls must require the JWT (`app/config/express/jwt.js`, see vulkano-backend-auth).

`GET /api/auth/current` is the backend's session-check endpoint — see vulkano-backend-auth for the `AuthController` convention it expects (`login`/`logout`/`current` method keys).

## After writing

- First non-root route in an entrypoint → its catch-all must be active in `app/config/routes.js` (`references/AGENTS/BACKEND.md` § Routes).
- Test hard-refresh on the new route, not just in-app navigation.

## Reference

`references/AGENTS/BACKEND.md` § Routes (backend catch-all), `references/AGENTS/FRONTEND.md` § Routes, vulkano-backend-auth (backend session-check convention).
