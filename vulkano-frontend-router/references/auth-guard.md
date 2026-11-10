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
