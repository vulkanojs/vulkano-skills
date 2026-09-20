---
name: vulkano-frontend-css
description: Use when styling a component/view in this Vulkano framework project's frontend/ — CSS Grid recommended default (no Flexbox unless PROJECT.md overrides it), the Foundation-style responsive grid system (`.row`/`.column`, unless PROJECT.md defines another), and BEM class naming.
---

# Frontend CSS

## Overview

Every component/view ships its own `_index.scss` (see vulkano-frontend-component § File & naming for the `.vue`/`.js`/`.scss` split). Class naming is BEM regardless of entrypoint or component size. Layout defaults to CSS Grid — check `PROJECT.md` first for a project-level override before assuming the default applies.

## When to use

- Writing or editing a component/view's `.scss` file
- Laying out a page/section in columns (responsive grid)
- Naming classes for a new component

Not for the component's `.vue`/`.js` file split or folder placement — see vulkano-frontend-component. Not for a pre-built UI-library component's own internal styles (`components/ui/`) — see vulkano-frontend-component § "Installed UI library is not optional" / `references/AGENTS/UI.md`.

## Layout — CSS Grid recommended, no Flexbox

Default: `display: grid` everywhere in `_index.scss`, never Flexbox. This is a recommendation, not framework-enforced — the layout system is a project decision like the UI kit (`references/AGENTS/UI.md`). Check `PROJECT.md` first: if the user explicitly picked Flexbox (or a mix) for this project, follow that instead and don't "fix" it back to Grid. No override documented → CSS Grid is the answer, don't ask again.

## Responsive grid system — `frontend/<entrypoint>/scss/_grid.scss`

Default: Foundation-style responsive grid, built on CSS Grid, imported once per entrypoint's `style.scss` (e.g. `frontend/website/style.scss`). Same override rule as above — check `PROJECT.md` first; a project can define its own grid system (different breakpoints, a Bootstrap-style grid, CSS Grid areas instead of a column count, etc.) instead of this one. No override documented → this is the grid system:

```html
<div class="row">
  <div class="column small-12 medium-6 large-4">...</div>
</div>
```

- `.row`: `display: grid; grid-template-columns: repeat(12, 1fr);` — 12-column grid.
- `.column`: `grid-column: span 12` default (mobile-first, full row).
- Size classes `.small-N` / `.medium-N` / `.large-N` / `.xlarge-N` (1-12), each `grid-column: span N` — `small` unscoped (base), `medium`/`large`/`xlarge` wrapped in `min-width` media queries (`$breakpoints` map: medium 40rem/640px, large 64rem/1024px, xlarge 75rem/1200px).
- Gutter: `0.875rem` (small), `0.9375rem` from `medium` up. `.row--collapsed` removes it (`gap: 0`).
- Nesting: any `.column` can also carry `.row` to nest a grid inside it — no special helper needed.
- No offset/push-pull classes (not needed yet — add only when a task requires them).

## Units — whole-pixel `rem` only

Allowed units: `rem`, `px`, `dvh`, `vw`, `%`. Never `em`, `vh` (use `dvh`), or `ch`.

A `rem` value is valid only if `value × 16` is a whole number of pixels (16px root). Check every `rem` you write:

| px  | rem         | px  | rem        | px  | rem      |
| --- | ----------- | --- | ---------- | --- | -------- |
| 1   | `0.0625rem` | 12  | `0.75rem`  | 24  | `1.5rem` |
| 2   | `0.125rem`  | 14  | `0.875rem` | 32  | `2rem`   |
| 4   | `0.25rem`   | 16  | `1rem`     | 40  | `2.5rem` |
| 6   | `0.375rem`  | 18  | `1.125rem` | 48  | `3rem`   |
| 8   | `0.5rem`    | 20  | `1.25rem`  | 64  | `4rem`   |
| 10  | `0.625rem`  | 22  | `1.375rem` | 80  | `5rem`   |

Wrong: `0.4rem` (6.4px), `0.9rem` (14.4px), `0.35rem` (5.6px), `1.1rem` (17.6px). Right: `0.375rem`, `0.875rem`, `0.375rem`, `1.125rem`. Pick the nearest row of the table instead of computing.

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

- New component's `_index.scss` → add its `@import './X/_index.scss';` line to the parent `_index.scss` aggregator (see vulkano-frontend-component § File & naming).
- Scan the `rem` values you wrote against the table above; any decimal-pixel value is a bug.
- Run `vp check` and `vp test`.

## Reference

`references/AGENTS/CSS.md` (same grid detail and unit rules as this skill). Project-level overrides (layout system, UI kit): `PROJECT.md` § Frontend conventions.
