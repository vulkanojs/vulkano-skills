---
name: vulkano-backend-templates
description: Use when creating or editing a server-rendered backend view, layout, partial, or error template under app/views in this Vulkano framework project (Nunjucks by default, or Handlebars if app/config/views.js sets it) — layout, partials, res.render locals, paginated lists, SEO meta block, i18n t(), custom formatting helpers/filters (dates, numbers, currency), and Vite asset injection.
---

# Backend Templates (Nunjucks / Handlebars)

Server-rendered pages under `app/views/`, rendered by a controller with `res.render('folder/file.html', { ...locals })`. Rules: `references/AGENTS/BACKEND.md` § Views. For `res.render` vs `res.vsr`, or `View.render` (PDF/email), see vulkano-backend-views.

**Engine:** `app/config/views.js` (or `app/config/views/config.js`) — absent or `engine: 'nunjucks'` → Nunjucks; `engine: 'handlebars'` → Handlebars. Check it first and use only that engine's syntax below.

## Before implementing

- Reuse a base layout from `app/views/_shared/templates/` — don't write a new `<html>` shell.
- Trace the controller action that renders the view: use only locals it passes (plus `res.locals`). Missing data → add it in the controller/model, never query from the template.
- Markup in 2+ views → `app/views/_shared/partials/` and include it — also when those views are written in the same task.

## Placement and naming

`app/views/` layout: `references/AGENTS/ARCHITECTURE.md`. View path mirrors the route: `/controller/action` → `app/views/controller/action.html`; module-nested controller → `app/views/module/controller/action.html`. `HomeController.get()` → `app/views/home/index.html`; `app/controllers/blog/PostsController.js` action `show` → `app/views/blog/posts/show.html`.

## Layout and partials

**Nunjucks** — the page extends a layout and fills its block; the layout declares `{% block content %}{% endblock %}`:

<!-- prettier-ignore -->
```html
{% extends "_shared/templates/default.html" %}
{% block content %}
  <h1>{{ title }}</h1>
  {% include "_shared/partials/oauth-brand.html" %}
{% endblock %}
```

**Handlebars** — wrapping is implicit: the page is only the content fragment; the `defaultLayout` (`_shared/templates/default.html`) injects it with triple-stash `{{{body}}}` (unescaped):

<!-- prettier-ignore -->
```html
<!-- _shared/templates/default.html -->
<html><head>...</head><body>{{{body}}}</body></html>

<!-- home/index.html -->
<h1>{{title}}</h1>
{{> oauth-brand}}
```

## Rendering locals and paginated lists

```js
res.render('orders/index.html', { pagination: await Order.getAll(req.query) });
```

`getAll` gives `{ items, page, totalPages, next, prev, … }` — iterate `.items` (bare array when `?page=all`, see `references/AGENTS/BACKEND.md` § Models; don't expose `page=all` on views with pagination controls).

<!-- prettier-ignore -->
```html
<!-- Nunjucks -->
{% for order in pagination.items %}<tr><td>{{ order.total }}</td></tr>{% endfor %}
{% if pagination.next %}<a href="?page={{ pagination.next }}">Next</a>{% endif %}

<!-- Handlebars -->
{{#each pagination.items}}<tr><td>{{this.total}}</td></tr>{{/each}}
{{#if pagination.next}}<a href="?page={{pagination.next}}">Next</a>{{/if}}
```

## SEO meta block (public layout)

Wiring and defaults: `references/AGENTS/BACKEND.md` § SEO wiring. The public layout prints:

<!-- prettier-ignore -->
```html
<!-- Nunjucks -->
<title>{{ seo.title }}</title>
<meta name="description" content="{{ seo.description }}" />
<link rel="canonical" href="{{ seo.url }}" />
{% if seo.noindex %}<meta name="robots" content="noindex, nofollow" />{% endif %}

<!-- Handlebars -->
<title>{{seo.title}}</title>
<meta name="description" content="{{seo.description}}" />
<link rel="canonical" href="{{seo.url}}" />
{{#if seo.noindex}}<meta name="robots" content="noindex, nofollow" />{{/if}}
```

A CMS/admin layout prints only `<title>` from `seo.title` (its controller sets `'VulkanoJS - Admin'`) plus a fixed `<meta name="robots" content="noindex, nofollow" />`.

## i18n

Built-in `t` helper (`node_modules/@vulkano/core/views/helpers/t.js`) — Nunjucks `{{ t('welcome') }}`, Handlebars `{{t "welcome"}}`. Logic beyond one `t()` call → pass `i18n.t(...)` as a local from the controller. Locale files: `app/config/locales/<lang>.js`, deep-merged with the core's `en`/`es`.

## Formatting helpers (date, number, currency)

No built-in formatter. One plain function per file, auto-loaded:

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

| Folder                      | Nunjucks                                                       | Handlebars                          |
| --------------------------- | -------------------------------------------------------------- | ----------------------------------- |
| `app/config/views/helpers/` | call only: `{{ currency(product.price) }}`                     | `{{currency product.price}}`        |
| `app/config/views/filters/` | pipe only: `{{ product.price \| currency }}` (value = 1st arg) | same as helpers (no filter concept) |

In Nunjucks a filter called as a function (or a helper piped) is undefined — pick the folder by how the template should read. Examples: `node_modules/@vulkano/core/examples/config/views/helpers/strpad.js`, `examples/config/views/filters/example.js`, core filters `vLowercase`/`vCamelCase`.

## Vite asset injection

`entry` = the area's key in `vite.entries.mjs` (`references/AGENTS/BUILD.md`). Style in `<head>`, script before `</body>`:

<!-- prettier-ignore -->
```html
<!-- Nunjucks: | safe so the tags aren't escaped -->
{{ vite({ entry: 'website', type: 'style' }) | safe }}
{{ vite({ entry: 'website', type: 'script' }) | safe }}

<!-- Handlebars: hash args + triple-stash -->
{{{vite entry="website" type="style"}}}
{{{vite entry="website" type="script"}}}
```

## After writing

- Public view → title/description per route (`references/AGENTS/BACKEND.md` § SEO wiring).
- Controller HTTP test asserts on the rendered HTML (vulkano-testing).
- Check it in a browser (`references/AGENTS/DEVTOOLS.md`) — `vp check` doesn't catch layout issues.

## Reference

`app/views/_shared/templates/default.html` and `admin.html`, `node_modules/@vulkano/core/bootstrap/engines/{nunjucks,handlebars}.js` (helper/filter/partials wiring), `node_modules/@vulkano/core/views/helpers/{vite,t}.js`, `node_modules/@vulkano/core/examples/config/views/config.js` (engine config).
