## Vanilla-JS entrypoint (server-rendered page, no Vue SPA)

For an area that's fully server-rendered (see vulkano-backend-templates) and only needs some compiled/bundled JS — a bit of interactivity, a third-party widget init, progressive enhancement — not a client-routed app:

```
frontend/<name>/
  app.js            # plain JS entry — no App.vue, no routes.js, no Api.js/Pinia unless this page actually calls the API
  style.scss        # this entry's own style entry, imported from app.js (or injected separately, project's call)
```

No `views/`/`components/`/`layouts/` folders unless the page's JS is genuinely split into multiple reusable pieces — most vanilla entries are one file. Skip the Vue-specific steps above: no `createWebHistory(base)` (there's no Vue Router), no `<div id="app">` mount point in the backend template, no frontend catch-all (the page is server-rendered per normal controller/view routing — see vulkano-backend-views — so a hard refresh always hits the real controller, not a client router).

Wiring is otherwise identical — same `vite.entries.mjs` key, same `@<folder>` alias, same `vite({ entry: '<name>', type: '...' })` calls in the backend template (see Wiring above), same HMR in dev. The only difference is what's on the other end of that bundle: a `createApp(App).mount('#app')` call for a Vue entry, vs. whatever plain DOM code this page needs for a vanilla one.
