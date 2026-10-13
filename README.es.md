# Vulkano Skills

Skills de Claude Code para proyectos construidos sobre el Vulkano Framework (stack Express + Mongoose + Vue 3, backend/frontend). Pensado para consumirse como submódulo git, así todos los proyectos Vulkano comparten las mismas convenciones y las actualizaciones salen desde un solo lugar.

## Qué hay aquí

Cada carpeta en la raíz del repo es un skill de Claude Code autocontenido (`SKILL.md` con frontmatter `name`/`description`). Cubren las convenciones backend y frontend del framework:

- `vulkano-backend-auth` — login/logout/verificación de sesión (modelo Auth/User dedicado, AuthController, JWT vía cookie httpOnly).
- `vulkano-backend-controller` — cableado de rutas, convención verbo HTTP a method-key, `res.vsr`/`res.render`, eventos socket.
- `vulkano-backend-model` — convenciones de schema Mongoose, CRUD, hooks, autopopulate.
- `vulkano-backend-views` — decidir entre vista HTML server-rendered vs respuesta JSON API.
- `vulkano-backend-templates` — layouts, partials, helpers/filters e inyección Vite para Nunjucks (por defecto) / Handlebars.
- `vulkano-frontend-component` — separación de archivos componente/vista Vue, Composition API, kit de UI instalado.
- `vulkano-frontend-css` — layout CSS Grid, sistema de grid responsive, naming BEM.
- `vulkano-frontend-entrypoint` — scaffold de nuevo entrypoint, app Vue o bundle JS vanilla para página server-rendered (CMS/admin, app adicional).
- `vulkano-frontend-form` — marcadores de campo requerido, validación solo JS, patrón fieldErrors.
- `vulkano-frontend-router` — rutas Vue Router, guards de auth, fetch de usuario actual.
- `vulkano-frontend-store` — Pinia store por concern, excepción `useAppStore`.
- `vulkano-template-update` — sincroniza los docs de agente de un proyecto (`AGENTS.md`, `references/AGENTS/`, skills) con el último template de Vulkano, migrando `PROJECT.md` sin tocar los valores del proyecto.
- `vulkano-testing` — layout Vitest, gate `TEST_MONGO_URI`, patrones de test por capa.

## Usar este repo como submódulo

Agrégalo a un proyecto Vulkano bajo `.claude/skills/`:

```bash
git submodule add https://github.com/vulkanojs/vulkano-skills.git .claude/skills/vulkano-skills
git submodule update --init --recursive
```

Claude Code descubre skills de forma recursiva bajo `.claude/skills/`, así que anidarlos un nivel más adentro (`vulkano-skills/`) no rompe el descubrimiento.

Para traer actualizaciones a un proyecto que ya tiene el submódulo:

```bash
git submodule update --remote --merge .claude/skills/vulkano-skills
```

## Skills vs `references/AGENTS/*.md`

Sin duplicación: `references/AGENTS/*.md` del proyecto (ARCHITECTURE, BACKEND, FRONTEND, BUILD…) guarda las reglas cortas; las skills de este repo guardan el código de ejemplo y el paso a paso. Una skill enlaza al doc de reglas en vez de repetirlo. Si este submódulo no está instalado, las reglas de `references/AGENTS/*.md` del proyecto bastan por sí solas.

## Agregar o editar un skill

1. Edita el `<nombre>/SKILL.md` correspondiente aquí, en este repo — no dentro del checkout del submódulo en un proyecto (los cambios ahí quedan locales a ese checkout hasta que les hagas push aquí y los traigas de vuelta).
2. Haz commit y push aquí.
3. En cada proyecto que lo consume, ejecuta `git submodule update --remote --merge` y haz commit del puntero del submódulo actualizado.

## Licencia

[MIT](LICENSE) © VulkanoJS
