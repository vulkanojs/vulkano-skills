# Vulkano Skills

Claude Code skills for projects built on the Vulkano Framework (Express + Mongoose + Vue 3 backend/frontend stack). Meant to be consumed as a git submodule so every Vulkano project shares the same conventions and updates to them roll out from one place.

## What's here

Each folder under `skills/` is a self-contained Claude Code skill (`SKILL.md` with frontmatter `name`/`description`). They cover the framework's backend and frontend conventions:

- `vulkano-backend-auth` — login/logout/session-check auth (dedicated Auth/User model, AuthController, JWT via httpOnly cookie).
- `vulkano-backend-controller` — route wiring, HTTP-verb-to-method-key convention, `res.vsr`/`res.render`, socket events.
- `vulkano-backend-model` — Mongoose schema/CRUD/hooks/autopopulate conventions.
- `vulkano-backend-views` — choosing server-rendered HTML vs JSON API response.
- `vulkano-backend-views-handlebars` — Handlebars layouts/partials/helpers.
- `vulkano-backend-views-nunjucks` — Nunjucks layouts/partials/filters (default engine).
- `vulkano-frontend-a11y` — accessibility minimums for images, nav, forms.
- `vulkano-frontend-analytics` — GA4/GTM tracking conventions.
- `vulkano-frontend-component` — Vue component/view file splitting, Composition API, BEM/Grid styling.
- `vulkano-frontend-entrypoint` — scaffolding a new Vue entrypoint (CMS/admin, extra app).
- `vulkano-frontend-form` — required-field markers, JS-only validation, fieldErrors pattern.
- `vulkano-frontend-router` — Vue Router routes, auth guards, current-user fetching.
- `vulkano-seo` — `res.locals.seo`, robots.txt/sitemap.xml, JSON-LD.
- `vulkano-testing` — Vitest layout, `TEST_MONGO_URI` gate, per-layer test patterns.

## Using this repo as a submodule

Add it to a Vulkano project under `.claude/skills/`:

```bash
git submodule add https://github.com/vulkanojs/vulkano-skills.git .claude/skills/vulkano-skills
git submodule update --init --recursive
```

Claude Code discovers skills recursively under `.claude/skills/`, so nesting them one level deeper inside `vulkano-skills/skills/` doesn't break discovery.

To pull updates into a project that already has the submodule:

```bash
git submodule update --remote --merge .claude/skills/vulkano-skills
```

## Adding or editing a skill

1. Edit the relevant `skills/<name>/SKILL.md` here, in this repo — not inside a project's submodule checkout (changes there are local to that checkout until pushed here and pulled back).
2. Commit and push here.
3. In each consuming project, run `git submodule update --remote --merge` and commit the updated submodule pointer.

## License

Internal use — not currently published for external consumption.
