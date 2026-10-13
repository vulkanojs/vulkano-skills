---
name: vulkano-backend-views-nunjucks
description: Use when creating or editing a server-rendered backend view, layout, partial, or error template in this Vulkano framework project, using Nunjucks under app/views (the default/most common engine) — layout inheritance, res.render, SEO locals, i18n, custom formatting helpers/filters (dates, numbers, currency), and Vite asset injection.
---

# Backend Views (Nunjucks)

## Overview

Backend views (`app/views/*.html`) are server-rendered templates — the crawlable surface for public pages (see `references/AGENTS/BACKEND.md` § SEO wiring). A controller renders one via `res.render('folder/file.html', { ...locals })`. This is separate from the Vue SPA (`frontend/`), which is client-rendered and not SEO-covered.

**Engine: check `app/config/views.js` first.** No file present, or `engine: 'nunjucks'`, means Nunjucks — this skill applies. If it sets `engine: 'handlebars'`, use the vulkano-backend-views-handlebars skill instead.

## When to use

- New page needs a server-rendered view/template
- Editing a layout, partial, or error page (404/500)
- Wiring SEO meta tags into a template
- Passing a translated string into a view
- Formatting a date, number, or currency value for display

Not for Vue components/views (`frontend/<entrypoint>/views/` — always its own entrypoint subfolder, e.g. `frontend/website/views/`, even with only 1) — see vulkano-frontend-component skill. Not for the controller that renders it — see vulkano-backend-controller skill.

## Before implementing

- Check `app/config/views.js` — if it sets `engine: 'handlebars'`, stop and switch to the vulkano-backend-views-handlebars skill.
- Check `app/views/_shared/templates/` for an existing base layout to extend instead of writing a new `<html>` shell.
- Check whether this view belongs to a public/crawlable area (needs the SEO block) or a CMS/admin area (doesn't) — see `references/AGENTS/ARCHITECTURE.md` § Entrypoints if the project splits front/CMS.
- Trace the controller action that renders this view — confirm what locals it actually passes (`res.render('x/y.html', { ...locals })`). Don't assume a variable exists in the template; if data is missing, add it in the controller/model, not with a query or fetch from inside the template.
- Markup repeated across 2+ views → extract to `app/views/_shared/partials/` and `{% include %}` it, instead of duplicating. This applies even when you're adding those 2+ views one at a time in the same task — check back against files you already wrote earlier in the SAME task, not just pre-existing ones. E.g. a branded side-panel/header block copy-pasted into `login.html`, then again into `error.html`, then again into a third view → stop, move it to `_shared/partials/oauth-brand.html` and include it in all three:
  ```html
  {% include "_shared/partials/oauth-brand.html" %}
  ```

## File placement

```
app/views/
  _shared/
    templates/     # base layouts, e.g. default.html
    partials/       # reusable includes
    errors/         # 404.html, 500.html
  <resource>/
    index.html      # rendered via res.render('<resource>/index.html', {...})
```

### Naming convention — mirror the URL/controller shape

Not automatic (every `res.render(...)` call is explicit), but keep the view path consistent with the route it serves, mirroring the routing convention from vulkano-backend-controller:

- `/controller/action` (`ControllerController.action`) → `app/views/controller/action.html`
- `/module/controller/action` (a controller nested under a module folder) → `app/views/module/controller/action.html`

E.g. `HomeController.get()` → `app/views/home/index.html` (`get` with no path tail renders the resource's default/index view). A controller in `app/controllers/blog/PostsController.js` rendering its `show` action → `app/views/blog/posts/show.html`.

## Layout inheritance

The page explicitly extends a base layout and fills its content block:

```html
{% extends "_shared/templates/default.html" %} {% block content %}
<!-- page content -->
{% endblock %}
```

The layout owns `<head>` (SEO meta, Vite asset tags) and declares `{% block content %}{% endblock %}` for pages to fill.

## Rendering from a controller

```js
res.render('home/index.html', { title: 'Home' });
```

Locals passed here, plus anything already on `res.locals` (e.g. set by a global middleware), are available in the template.

## Rendering a paginated list / report

`Model.getAll(props)` (see vulkano-backend-model) resolves to a page object: `{ items, page, perPage, totalPages, next, prev, cursor }` — pass it straight through as a local and iterate `.items`, then use the rest for pagination controls:

```js
// controller
res.render('orders/index.html', { pagination: await Order.getAll(req.query) });
```

```html
<!-- template -->
{% for order in pagination.items %}
<tr><td>{{ order.total }}</td></tr>
{% endfor %} {% if pagination.prev %}<a href="?page={{ pagination.prev }}">Prev</a>{% endif %}
<span>{{ pagination.page }} / {{ pagination.totalPages }}</span>
{% if pagination.next %}<a href="?page={{ pagination.next }}">Next</a>{% endif %}
```

**Gotcha**: if this endpoint's `?page=` can be `all`, `getAll` resolves to a bare array instead — no `.items`, no `.page`/`.next`/`.prev` (see vulkano-backend-model § `getAll(props)` resolves to one of TWO different shapes). A view built for a normal paginated page will break (`pagination.items` is `undefined`) if a request hits it with `?page=all`. Either don't expose `page=all` on views that render pagination controls, or normalize the shape in the controller before calling `res.render`.

## SEO meta (public/crawlable views only)

Base layout prints `res.locals.seo.*`, set by `app/config/middlewares/seo.js` and overridable per-route in the controller before `res.render`:

```html
<title>{{ seo.title }}</title>
<meta name="description" content="{{ seo.description }}" />
<link rel="canonical" href="{{ seo.url }}" />
{% if seo.noindex %}<meta name="robots" content="noindex, nofollow" />{% endif %}
```

Defaults, per-route override and private-mode noindex: `references/AGENTS/BACKEND.md` § SEO wiring. A CMS/admin layout skips this block: only the `<title>` from `seo.title` (its controller sets `'VulkanoJS - Admin'`, any admin controller may override) plus a fixed `<meta name="robots" content="noindex, nofollow" />` — no description/canonical/og.

## i18n

A `t(key, options)` helper is registered out of the box (`node_modules/@vulkano/core/views/helpers/t.js`). Call it directly, no controller wiring needed:

```html
{{ t('welcome') }}
```

Fall back to passing a translated string as a local from the controller only if a view needs translation logic more complex than a single `t()` call:

```js
res.render('home/index.html', { welcome: i18n.t('welcome') });
```

```html
{{ welcome }}
```

Locale files: `app/config/locales/<lang>.js`, keyed by filename, deep-merged with the core's own `en`/`es` defaults.

## Custom helpers/filters — date/number/currency formatting

Not built in — Vulkano ships no formatter (no `date`/`currency`/`number` filter). Write one as a plain function and drop it in the right folder; it's auto-loaded, no registration code:

- `app/config/views/helpers/<name>.js` — registered as a global: call syntax only, `{{ name(value, args) }}`.
- `app/config/views/filters/<name>.js` — registered as a filter: pipe syntax only, `{{ value | name(args) }}` (the piped value is the first argument).

Pick the folder by how the template should read — a filter called as `{{ name(x) }}` (or a helper piped) fails as undefined.

```js
// app/config/views/helpers/currency.js
module.exports = (amount, currency = 'USD') =>
  new Intl.NumberFormat('es-MX', { style: 'currency', currency }).format(amount || 0);
```

```js
// app/config/views/helpers/date.js
module.exports = (value, format = 'es-MX') =>
  value ? new Intl.DateTimeFormat(format, { dateStyle: 'medium' }).format(new Date(value)) : '';
```

Usage:

```html
{{ currency(product.price) }} {{ date(order.createdAt) }}
```

Reference implementations already in this convention: `node_modules/@vulkano/core/examples/config/views/helpers/strpad.js` (multi-arg helper), `examples/config/views/filters/example.js` (filter-style), and the core's own always-loaded `vLowercase`/`vCamelCase` filters (`node_modules/@vulkano/core/views/filters/`). One file per formatter — don't bundle unrelated formatting logic into a single catch-all helper.

## Vite asset injection

Function-call syntax, pipe through `| safe` so the HTML isn't escaped:

```html
{{ vite({ entry: 'app', type: 'style' }) | safe }}
<!-- in <head> -->
{{ vite({ entry: 'app', type: 'script' }) | safe }}
<!-- before </body> -->
```

`entry` must match a key in `vite.entries.mjs`'s `entries` map — use the entry name for the area this view belongs to (front vs CMS, see `references/AGENTS/BUILD.md`).

## After writing

- New public/crawlable view → meta/sitemap wiring per `references/AGENTS/BACKEND.md` § SEO wiring.
- Controller HTTP test for this view asserts on the rendered HTML body (see vulkano-testing).
- Visual check in a browser (chrome-devtools MCP per `references/AGENTS/DEVTOOLS.md`) — layout/styling issues aren't caught by `vp check`.

## Handoff

Report: view file(s) created/changed, controller-action it's rendered from, layout/partials reused, SEO/i18n applied (or explicitly not needed), and any locals the controller still needs to add (unresolved data).

## Reference

`references/AGENTS/BACKEND.md`, `references/AGENTS/BUILD.md`, `app/views/_shared/templates/default.html` (reference layout), `app/config/views.js` (engine override, if present), `node_modules/@vulkano/core/bootstrap/engines/nunjucks.js` (globals/filters/helpers wiring), `node_modules/@vulkano/core/views/helpers/vite.js` and `views/helpers/t.js` (built-in helper implementations), `node_modules/@vulkano/core/examples/config/views/config.js` (engine override template), `node_modules/@vulkano/core/examples/config/views/helpers/strpad.js` and `examples/config/views/filters/example.js` (custom helper/filter templates).
