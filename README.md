# Vulkano Skills

Claude Code skills for projects built on the Vulkano Framework (Express + Mongoose + Vue 3 backend/frontend stack). Meant to be consumed as a git submodule so every Vulkano project shares the same conventions and updates to them roll out from one place.

## What's here

Each folder at the repo root is a self-contained Claude Code skill (`SKILL.md` with frontmatter `name`/`description`). They cover the framework's backend and frontend conventions:

- `vulkano-backend-auth` — login/logout/session-check auth (dedicated Auth/User model, AuthController, JWT via httpOnly cookie).
- `vulkano-backend-controller` — route wiring, HTTP-verb-to-method-key convention, `res.vsr`/`res.render`, socket events.
- `vulkano-backend-model` — Mongoose schema/CRUD/hooks/autopopulate conventions.
- `vulkano-backend-views` — choosing server-rendered HTML vs JSON API response.
- `vulkano-backend-views-handlebars` — Handlebars layouts/partials/helpers.
- `vulkano-backend-views-nunjucks` — Nunjucks layouts/partials/filters (default engine).
- `vulkano-frontend-a11y` — accessibility minimums for images, nav, forms.
- `vulkano-frontend-analytics` — GA4/GTM tracking conventions.
- `vulkano-frontend-component` — Vue component/view file splitting, Composition API, installed UI kit.
- `vulkano-frontend-css` — CSS Grid layout, responsive grid system, BEM naming.
- `vulkano-frontend-entrypoint` — scaffolding a new entrypoint, Vue app or vanilla-JS bundle for a server-rendered page (CMS/admin, extra app).
- `vulkano-frontend-form` — required-field markers, JS-only validation, fieldErrors pattern.
- `vulkano-frontend-router` — Vue Router routes, auth guards, current-user fetching.
- `vulkano-frontend-store` — Pinia store-per-concern, `useAppStore` app-shell exception.
- `vulkano-seo` — `res.locals.seo`, robots.txt/sitemap.xml, JSON-LD.
- `vulkano-testing` — Vitest layout, `TEST_MONGO_URI` gate, per-layer test patterns.

## Using this repo as a submodule

Add it to a Vulkano project under `.claude/skills/`:

```bash
git submodule add https://github.com/vulkanojs/vulkano-skills.git .claude/skills/vulkano-skills
git submodule update --init --recursive
```

Claude Code discovers skills recursively under `.claude/skills/`, so nesting them one level deeper inside `vulkano-skills/` doesn't break discovery.

To pull updates into a project that already has the submodule:

```bash
git submodule update --remote --merge .claude/skills/vulkano-skills
```

## Duplication with `references/AGENTS/*.md` is intentional

Some mandatory content (e.g. `ENTRYPOINTS.md` wiring steps) is deliberately repeated in both a consuming project's `references/AGENTS/*.md` and the matching skill here. This submodule can be removed or left uninstalled in a given project, so the project's own `references/AGENTS/*.md` must stay mandatory and self-sufficient on its own — it can't depend on this submodule for critical info. Skills here are the detailed/actionable version; `references/AGENTS/*.md` is the fallback. Don't collapse that duplication into a pointer.

## Adding or editing a skill

1. Edit the relevant `<name>/SKILL.md` here, in this repo — not inside a project's submodule checkout (changes there are local to that checkout until pushed here and pulled back).
2. Commit and push here.
3. In each consuming project, run `git submodule update --remote --merge` and commit the updated submodule pointer.

## License

[MIT](LICENSE) © VulkanoJS
