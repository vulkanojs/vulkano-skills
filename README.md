# Vulkano Skills

Claude Code skills for projects built on the Vulkano Framework (Express + Mongoose + Vue 3 backend/frontend stack). Meant to be consumed as a git submodule so every Vulkano project shares the same conventions and updates to them roll out from one place.

## What's here

Each folder at the repo root is a self-contained Claude Code skill (`SKILL.md` with frontmatter `name`/`description`). They cover the framework's backend and frontend conventions:

- `vulkano-backend-auth` — login/logout/session-check auth (dedicated Auth/User model, AuthController, JWT via httpOnly cookie).
- `vulkano-backend-controller` — route wiring, HTTP-verb-to-method-key convention, `res.vsr`/`res.render`, socket events.
- `vulkano-backend-model` — Mongoose schema/CRUD/hooks/autopopulate conventions.
- `vulkano-backend-views` — choosing server-rendered HTML vs JSON API response.
- `vulkano-backend-templates` — Nunjucks (default) / Handlebars layouts, partials, helpers/filters, Vite injection.
- `vulkano-frontend-component` — Vue component/view file splitting, Composition API, installed UI kit.
- `vulkano-frontend-css` — CSS Grid layout, responsive grid system, BEM naming.
- `vulkano-frontend-entrypoint` — scaffolding a new entrypoint, Vue app or vanilla-JS bundle for a server-rendered page (CMS/admin, extra app).
- `vulkano-frontend-form` — required-field markers, JS-only validation, fieldErrors pattern.
- `vulkano-frontend-router` — Vue Router routes, auth guards, current-user fetching.
- `vulkano-frontend-store` — Pinia store-per-concern, `useAppStore` app-shell exception.
- `vulkano-template-update` — sync a project's agent docs (`AGENTS.md`, `references/AGENTS/`, skills) from the latest Vulkano template, migrating `PROJECT.md` without touching project values.
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

## Skills vs `references/AGENTS/*.md`

No duplication: a consuming project's `references/AGENTS/*.md` (ARCHITECTURE, BACKEND, FRONTEND, BUILD…) holds the short rules; the skills here hold the worked code and step-by-step how-to. A skill links to the rule doc instead of restating it. If this submodule isn't installed, the project's `references/AGENTS/*.md` rules are enough on their own.

## Adding or editing a skill

1. Edit the relevant `<name>/SKILL.md` here, in this repo — not inside a project's submodule checkout (changes there are local to that checkout until pushed here and pulled back).
2. Commit and push here.
3. In each consuming project, run `git submodule update --remote --merge` and commit the updated submodule pointer.

## License

[MIT](LICENSE) © VulkanoJS
