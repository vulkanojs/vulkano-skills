---
name: vulkano-testing
description: Use when writing or reviewing a test for a controller, model, service, middleware, socket handler, Pinia store, or script in this Vulkano framework project — Vitest config, TEST_MONGO_URI gate, directory layout mirroring source, waitForReady/dbCleanup helpers, and per-layer test patterns.
---

# Testing

## Overview

Runner is Vitest via `vp test`. Every new/changed controller, model, service, or middleware gets a test — this isn't optional per project convention.

## When to use

Any task that creates/edits `app/controllers/`, `app/models/`, `app/services/`, `app/config/middlewares/`, `app/config/sockets/`, `frontend/<entrypoint>/store/`, or `frontend/<entrypoint>/composables/`/`utils/` code — after writing the code, write/update its test before considering the task done.

## Database — `TEST_MONGO_URI`, optional

**No fallback to `MONGO_URI`, ever** — `test/helpers/bootstrap.js` deletes `process.env.MONGO_URI` when `TEST_MONGO_URI` isn't set, so a test suite never connects to the dev/prod database. Without it, `@vulkano/core`'s `loadDatabaseApplication()` sees a falsy `connection` and skips `mongoose.connect()` entirely (`node_modules/@vulkano/core/database/mongodb.js`) — the app still boots (`waitForReady()` resolves), models/controllers with no DB-dependent logic still testable, only DB reads/writes fail. Set `TEST_MONGO_URI` in `.env` to a dedicated test database — never the same one as `MONGO_URI` — as soon as a model, or anything that reads/writes one, needs a real test.

## Directory layout — mirror the source tree

One `test/<source-folder>/` per top-level source folder, one layer subfolder per architectural layer inside it — never merge two source folders' tests into one:

```
test/
  helpers/                    bootstrap.js (waitForReady), dbCleanup.js (clearCollections), test<Model>.js factories
  app/
    models/<Name>.test.js
    controllers/<Name>.http.test.js
    services/<Name>.test.js
    middlewares/<Name>.test.js
    integration/<Flow>.test.js
  <script>.test.js            standalone scripts/*.js not under app/
  frontend/
    website/
      store/<name>.test.js
      composables/<name>.test.js
      utils/<name>.test.js
      integration/<Flow>.test.js
    admin/
      store/<name>.test.js
      composables/<name>.test.js
      utils/<name>.test.js
      integration/<Flow>.test.js
```

Mirrors `frontend/`'s own convention (see vulkano-frontend-entrypoint): `test/frontend/` is always a container, one subfolder per entrypoint, matching `frontend/<entrypoint>/` exactly — never `test/admin/` as a root-level sibling.

## Patterns by test type

| Type                     | Path                                                       | Pattern                                                                                                                                                                                                                                                                                                                          |
| ------------------------ | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Model                    | `test/app/models/*.test.js`                                | `beforeAll(() => waitForReady())`, `afterEach(() => dbCleanup.clearCollections('Name'))`, assert business rules directly (`.rejects.toThrow(...)` for invalid input)                                                                                                                                                             |
| Controller/HTTP          | `test/app/controllers/*.http.test.js`                      | Same setup; hit the real running app with native `fetch` against `http://localhost:${process.env.PORT}` — no `supertest`. API: assert `res.vsr()` envelope `{ success, statusCode, data }`. View: assert rendered HTML body                                                                                                      |
| Service                  | `test/app/services/*.test.js`                              | Same boot/mock pattern, call the function directly (no HTTP)                                                                                                                                                                                                                                                                     |
| Middleware               | `test/app/middlewares/*.test.js`                           | Unit-test with mock `req`/`res`/`next` for pure logic, OR verify end-to-end through a controller/HTTP test                                                                                                                                                                                                                       |
| Integration              | `test/app/integration/*.test.js`                           | Full business flow across models (signup → login → protected route); factory helpers from `test/helpers/`; clear every touched collection in dependency order                                                                                                                                                                    |
| Script                   | `test/<script>.test.js`                                    | Plain unit tests, no boot/DB, no `TEST_MONGO_URI` needed unless the script itself touches a model                                                                                                                                                                                                                                |
| Frontend store           | `test/frontend/<entrypoint>/store/*.test.js`               | No app boot/DB; `createPinia()` + `setActivePinia()` in `beforeEach`; inject a mock `$api` — never hit real network; shim browser globals or mark `// @vitest-environment jsdom` if a real DOM is needed                                                                                                                         |
| Frontend composable/util | `test/frontend/<entrypoint>/{composables,utils}/*.test.js` | Plain unit test, no app boot/DB, no Pinia, no `jsdom` unless the composable touches the DOM; call the exported function(s) directly with fixture input, assert the returned value (e.g. `V.required('msg')('')` → `'msg'`; `useFormValidation(model, rules).validate(cb)` → `cb` called with `isValid`, `fieldErrors` populated) |

Mock outbound external calls (`ApiClient`, third-party APIs) with `vi.spyOn(...).mockResolvedValue(...)`, restored in `afterEach` — never hit a real third-party endpoint from a test.

## Writing a new test

1. Put it under `test/<source-folder>/<layer>/`, mirroring the source path.
2. Reuse `waitForReady()` and `dbCleanup.clearCollections(...)` instead of duplicating boot/cleanup logic. Add a `test/helpers/test<Model>.js` factory (`makeUser(overrides)`-style) if the model needs one.
3. Clear every collection the test writes to, in `afterEach` — `isolate: false`/`fileParallelism: false` mean all files share one DB connection, so leftovers leak into the next file.
4. Mock outbound calls, restore in `afterEach`.

## Running

```
vp test          # full suite once — runs without a DB if TEST_MONGO_URI unset (DB-dependent tests will fail)
vp test watch    # watch mode
```

No coverage threshold or CI script configured — don't claim one exists without checking `package.json`/`vite.config.mjs` first.

## Browser E2E — not a CI suite

No Playwright/Cypress installed. For end-to-end verification of a user flow, drive a real browser with the Playwright MCP or chrome-devtools MCP (navigate, click, fill, screenshot, console/network) as part of verifying the change — agent-driven verification for the task at hand, not a regression suite (see `references/AGENTS/DEVTOOLS.md`).

## After writing

- Run `vp check` and `vp test` before considering the change done.
- New model/controller/service/middleware with no test yet → write one now, don't defer.

## Reference

`references/AGENTS/TESTING.md` (full detail), `test/app/controllers/Home.http.test.js` (existing worked example), vulkano-backend-model, vulkano-backend-controller, vulkano-backend-auth (auth-flow test coverage: `req.auth`, JWT cookie).
