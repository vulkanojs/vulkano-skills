# Vulkano Skills

Skills de Claude Code para proyectos construidos sobre el Vulkano Framework (stack Express + Mongoose + Vue 3, backend/frontend). Pensado para consumirse como submódulo git, así todos los proyectos Vulkano comparten las mismas convenciones y las actualizaciones salen desde un solo lugar.

## Qué hay acá

Cada carpeta en la raíz del repo es un skill de Claude Code autocontenido (`SKILL.md` con frontmatter `name`/`description`). Cubren las convenciones backend y frontend del framework:

- `vulkano-backend-auth` — login/logout/verificación de sesión (modelo Auth/User dedicado, AuthController, JWT vía cookie httpOnly).
- `vulkano-backend-controller` — cableado de rutas, convención verbo HTTP a method-key, `res.vsr`/`res.render`, eventos socket.
- `vulkano-backend-model` — convenciones de schema Mongoose, CRUD, hooks, autopopulate.
- `vulkano-backend-views` — decidir entre vista HTML server-rendered vs respuesta JSON API.
- `vulkano-backend-views-handlebars` — layouts/partials/helpers Handlebars.
- `vulkano-backend-views-nunjucks` — layouts/partials/filters Nunjucks (motor por defecto).
- `vulkano-frontend-a11y` — mínimos de accesibilidad para imágenes, navegación, forms.
- `vulkano-frontend-analytics` — convenciones de tracking GA4/GTM.
- `vulkano-frontend-component` — separación de archivos componente/vista Vue, Composition API, kit de UI instalado.
- `vulkano-frontend-css` — layout CSS Grid, sistema de grid responsive, naming BEM.
- `vulkano-frontend-entrypoint` — scaffold de nuevo entrypoint, app Vue o bundle JS vanilla para página server-rendered (CMS/admin, app adicional).
- `vulkano-frontend-form` — marcadores de campo requerido, validación solo JS, patrón fieldErrors.
- `vulkano-frontend-router` — rutas Vue Router, guards de auth, fetch de usuario actual.
- `vulkano-frontend-store` — Pinia store por concern, excepción `useAppStore`.
- `vulkano-seo` — `res.locals.seo`, robots.txt/sitemap.xml, JSON-LD.
- `vulkano-template-update` — sincroniza los docs de agente de un proyecto (`AGENTS.md`, `references/AGENTS/`, skills) con el último template de Vulkano, migrando `PROJECT.md` sin tocar los valores del proyecto.
- `vulkano-testing` — layout Vitest, gate `TEST_MONGO_URI`, patrones de test por capa.

## Usar este repo como submódulo

Agregalo a un proyecto Vulkano bajo `.claude/skills/`:

```bash
git submodule add https://github.com/vulkanojs/vulkano-skills.git .claude/skills/vulkano-skills
git submodule update --init --recursive
```

Claude Code descubre skills de forma recursiva bajo `.claude/skills/`, así que anidarlos un nivel más adentro (`vulkano-skills/`) no rompe el descubrimiento.

Para traer actualizaciones a un proyecto que ya tiene el submódulo:

```bash
git submodule update --remote --merge .claude/skills/vulkano-skills
```

## La duplicación con `references/AGENTS/*.md` es intencional

Cierto contenido mandatorio (ej. `ENTRYPOINTS.md`, pasos de wiring) se repite a propósito tanto en `references/AGENTS/*.md` del proyecto consumidor como en el skill correspondiente acá. Este submódulo puede eliminarse o no estar instalado en un proyecto dado, así que `references/AGENTS/*.md` del proyecto debe quedar mandatorio y autosuficiente por sí solo — no puede depender de este submódulo para info crítica. Los skills acá son la versión detallada/accionable; `references/AGENTS/*.md` es el respaldo. No colapsar esa duplicación a un puntero.

## Agregar o editar un skill

1. Editá el `<nombre>/SKILL.md` correspondiente acá, en este repo — no dentro del checkout del submódulo en un proyecto (cambios ahí quedan locales a ese checkout hasta pushearlos acá y traerlos de vuelta).
2. Commiteá y pusheá acá.
3. En cada proyecto que lo consume, corré `git submodule update --remote --merge` y commiteá el puntero de submódulo actualizado.

## Licencia

[MIT](LICENSE) © VulkanoJS
