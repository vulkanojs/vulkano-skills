---
name: vulkano-template-update
description: Use when the user asks to update, sync, or refresh this project's agent docs (AGENTS.md, references/AGENTS/, skills) from the latest Vulkano template — overwrites template-owned docs, migrates PROJECT.md without touching the project's own values.
---

# Vulkano Template Update

## Overview

A project created from the Vulkano template keeps its agent docs in a few places that the template owns and a few that the project owns. This skill brings the template-owned ones up to date and migrates the project-owned ones by structure only, so pulling a newer template never loses project decisions.

## When to use

The user says "update/sync the docs/references/skills from the template", "get this project up to date with Vulkano", or similar.

Not for updating app code, `frontend/`/`app/`, or dependencies other than `@vulkano/core` — those are project-owned and never touched here.

## Ownership

- **Template-owned** (overwrite from the template): `AGENTS.md` (`CLAUDE.md` is a symlink to it), `references/` (`references/AGENTS/*.md` and `references/CHANGELOG.md`), `.claude/skills/vulkano-skills` (git submodule).
- **Project-owned** (never overwrite, only migrate): `PROJECT.md`, `CHANGELOG.md` (the project's own log), `README.md`, `.env`, `app/`, `frontend/`, `package.json`. The one exception: the `@vulkano/core` version (step 8).

## Steps

Follow in order. Stop and ask before anything not listed. Never run `git commit` — propose a message at the end.

1. `git status` must be clean; otherwise stop. Create branch `chore/template-docs-sync`.
2. Skills: `git submodule update --init --remote --merge .claude/skills/vulkano-skills`. Report old → new commit. If this skill itself changed, re-read it before continuing.
3. Template: add remote `vulkano-template` = `https://github.com/vulkanojs/vulkano.git` (skip if present), then `git fetch vulkano-template`.
4. Read both changelogs. The template's: `git show vulkano-template/master:references/CHANGELOG.md` — what changed upstream and which migrations project-owned files need. This project's `CHANGELOG.md` — what the project changed: local edits to template-owned files, migrations already done, previous `Vulkano template sync <sha>` entries. Interpret them together: skip migrations the project log shows as done or whose condition is already satisfied, and treat logged local edits or deviations as intentional (ask before overwriting them).
5. Diff the template-owned paths (`AGENTS.md`, `references`). Base: the sha in the project's latest `Vulkano template sync <sha>` entry; if none, `git merge-base HEAD vulkano-template/master` when the project shares history with the template; otherwise `HEAD`. Report every project-side difference in `AGENTS.md` that the project log doesn't explain as a possible local edit (project facts belong in `PROJECT.md`, not `AGENTS.md`).
6. Show the user a summary: files added / removed / changed. Then apply: `git checkout vulkano-template/master -- AGENTS.md references`, and list (then delete) files under `references/AGENTS` that no longer exist in the template.
7. Migrate project-owned files using the changelog's **Migration** entries. Each is a condition plus an action: check the condition against the project's current state and skip it if already satisfied. Keep this project's values; change structure only. Always check:
   - `PROJECT.md`: the SEO/Analytics/Accessibility table is the only place that decides per-area opt-outs — keep all rows.
   - `PROJECT.md` § Deployment: CI/CD line plus a per-environment table listing only the mechanism(s) this project uses. If it still holds the old pasted catalog, replace it with rows for what the project actually uses (infer from `Dockerfile`, `docker-compose.yml`, `ecosystem.config.js`, `nixpacks.toml`, then ask the user to confirm).
   - `.env.example`: compare with the template's, add missing variables and comments. Never touch `.env`.
8. Core: update `@vulkano/core`.
   - Current: the version installed in `node_modules/@vulkano/core/package.json` and the range where the project pins it (`catalog['@vulkano/core']` in `pnpm-workspace.yaml`, or `dependencies` in `package.json` if the project doesn't use a catalog). Latest: `pnpm view @vulkano/core version`.
   - If a newer version exists, read the core's changelog for the versions in between (`CHANGELOG.md` in the package, or the core repo's releases) and tell the user what changed. A major bump, or a breaking change that touches `app/config/`, controllers, or models: stop and ask before installing.
   - Otherwise set the new range (`^<latest>`) where the project pins it and run `pnpm install`. Package and lockfile changes are security-sensitive (see `AGENTS.md`): show the diff, and only touch `@vulkano/core`, no other dependency.
   - If the changelog asks for changes in project code, list them for the user; don't apply them silently.
9. Verify: run `vp check` and `vp test`. Check that every relative link and anchor in `AGENTS.md`, `PROJECT.md` and `references/AGENTS/*.md` resolves. Grep the project for references to removed files (e.g. `DEPLOYMENT.md`, `ARCHITECTURE.md § Multiple entry points`).
10. Log: add an entry to the project's `CHANGELOG.md`, titled `## <date> — Vulkano template sync <vulkano-template/master sha>`, listing files added/removed/changed, migrations applied and skipped, and core version old → new.
11. Report: the same summary plus anything ambiguous, and a suggested commit message in the form `[chore] docs: sync agent docs with Vulkano template <sha>`.

## First run in an older project

A project whose submodule predates this skill doesn't have it yet. Run once by hand, then ask again:

```bash
git submodule update --init --remote --merge .claude/skills/vulkano-skills
```
