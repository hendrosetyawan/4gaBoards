# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Non-Obvious Documentation Context

- **`packages/locales/`** is not just locale JSON — it exports JS modules with named exports per language; the client's i18n test (`client/src/tests/i18n.test.js`) reads directly from the built `@4gaboards/locales` dist to verify coverage.
- **Sails blueprints are disabled** — the server has no auto-generated REST routes. All routes and policies are explicit in `server/api/controllers/`, `server/api/policies/`, and `server/config/`.
- **`server/db/`** holds Knex migration files; Sails ORM (`sails-hook-orm`) handles model-level queries while Knex handles raw schema migrations — they coexist.
- **ESLint configs are shared from root** — `eslint-configs/base.mjs`, `eslint-configs/client.mjs`, `eslint-configs/plugins.mjs` are used by all sub-packages. Do not duplicate config in individual packages.
- **`pnpm-workspace.yaml` includes `skills: dirs: - .claude/skills`** — this is a workspace-level configuration for project-specific skills (not standard pnpm workspace syntax).
- **`client/` is not CRA-ejected** — it uses custom build scripts in `client/scripts/` (build.js, start.js, test.js) that wrap webpack/jest; there's no `react-scripts` dependency.
- **Playwright tests live in `tests/`** (separate workspace package), not in `client/` or `server/`.
