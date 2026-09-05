# Vulkano Skills

Skills de Claude Code para proyectos construidos sobre el [Vulkano Framework](https://github.com/argordmel/vulkano-framework) (stack Express + Mongoose + Vue 3, backend/frontend). Pensado para consumirse como submódulo git, así todos los proyectos Vulkano comparten las mismas convenciones y las actualizaciones salen desde un solo lugar.

## Qué hay acá

Cada carpeta bajo `skills/` es un skill de Claude Code autocontenido (`SKILL.md` con frontmatter `name`/`description`). Cubren las convenciones backend y frontend del framework:

- `vulkano-backend-auth` — login/logout/verificación de sesión (modelo Auth/User dedicado, AuthController, JWT vía cookie httpOnly).
- `vulkano-backend-controller` — cableado de rutas, convención verbo HTTP a method-key, `res.vsr`/`res.render`, eventos socket.
- `vulkano-backend-model` — convenciones de schema Mongoose, CRUD, hooks, autopopulate.
- `vulkano-backend-views` — decidir entre vista HTML server-rendered vs respuesta JSON API.
- `vulkano-backend-views-handlebars` — layouts/partials/helpers Handlebars.
- `vulkano-backend-views-nunjucks` — layouts/partials/filters Nunjucks (motor por defecto).
- `vulkano-frontend-a11y` — mínimos de accesibilidad para imágenes, navegación, forms.
- `vulkano-frontend-analytics` — convenciones de tracking GA4/GTM.
- `vulkano-frontend-component` — separación de archivos componente/vista Vue, Composition API, estilos BEM/Grid.
- `vulkano-frontend-entrypoint` — scaffold de nuevo entrypoint Vue (CMS/admin, app adicional).
- `vulkano-frontend-form` — marcadores de campo requerido, validación solo JS, patrón fieldErrors.
- `vulkano-frontend-router` — rutas Vue Router, guards de auth, fetch de usuario actual.
- `vulkano-seo` — `res.locals.seo`, robots.txt/sitemap.xml, JSON-LD.
- `vulkano-testing` — layout Vitest, gate `TEST_MONGO_URI`, patrones de test por capa.

## Usar este repo como submódulo

Agregalo a un proyecto Vulkano bajo `.claude/skills/`:

```bash
git submodule add https://github.com/argordmel/vulkano-skills.git .claude/skills/vulkano-skills
git submodule update --init --recursive
```

Claude Code descubre skills de forma recursiva bajo `.claude/skills/`, así que anidarlos un nivel más adentro (`vulkano-skills/skills/`) no rompe el descubrimiento.

Para traer actualizaciones a un proyecto que ya tiene el submódulo:

```bash
git submodule update --remote --merge .claude/skills/vulkano-skills
```

## Agregar o editar un skill

1. Editá el `skills/<nombre>/SKILL.md` correspondiente acá, en este repo — no dentro del checkout del submódulo en un proyecto (cambios ahí quedan locales a ese checkout hasta pushearlos acá y traerlos de vuelta).
2. Commiteá y pusheá acá.
3. En cada proyecto que lo consume, corré `git submodule update --remote --merge` y commiteá el puntero de submódulo actualizado.

## Licencia

Uso interno — no publicado actualmente para consumo externo.
