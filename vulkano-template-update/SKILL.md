---
name: vulkano-template-update
description: Use when the user asks to update, sync, or refresh this project's agent docs (AGENTS.md, references/AGENTS/, skills) or @vulkano/core from the latest Vulkano template — runs `npx @vulkano/cli update`, then applies the migrations it lists and reviews kept local edits. Never edits PROJECT.md values.
---

# Vulkano Template Update

## Overview

`npx @vulkano/cli update` does the mechanical sync: it overwrites the template-owned docs (`AGENTS.md`, `CLAUDE.md`, `references/`), pins the skills submodule, creates `PROJECT.md` only if missing, adds new `.env.example` keys, bumps `@vulkano/core`, and writes `.vulkano.json` plus a `CHANGELOG.md` entry. This skill covers what needs judgment afterwards.

## When to use

The user says "update/sync the docs/references/skills from the template", "get this project up to date with Vulkano", or similar. Not for app code changes unrelated to a template migration.

## Steps

1. Run `npx @vulkano/cli update` in the project root, without `--yes`: the user answers the per-file questions about local edits. If it stops (dirty tree, no network, rate limit), report the message and stop.
2. Migrations: the report lists **Migration** entries from the template's `references/CHANGELOG.md`. Each is a condition plus an action. Check the condition against the project; skip it if already satisfied, apply it otherwise, and ask when in doubt. Never change `PROJECT.md` values; only its structure when a migration says so. On a first sync the list covers the last 90 days only: check older entries in `references/CHANGELOG.md` against the project's real state.
3. Kept local edits: for each file the user kept, check if it holds project facts. If so, propose moving them to `PROJECT.md` so the next sync can overwrite the file.
4. Verify: `vp check` and `vp test`; every relative link and anchor in `AGENTS.md`, `PROJECT.md` and `references/AGENTS/*.md` resolves; grep the project (`app/`, `frontend/`, `README.md`, `public/`, `test/`) for references to files the sync removed.
5. Report what the CLI changed, the migrations applied/skipped, and the suggested commit message. Never run `git commit`.
