# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Stack

- **Monorepo** (pnpm workspaces): `client/` (React + Webpack + Redux-ORM + Redux-Saga), `server/` (Sails.js v1 + Knex + PostgreSQL + Redis), `tests/` (Playwright), `packages/enums`, `packages/locales`, `packages/utils`
- Package manager: **pnpm** (v12+, Node v24.11 required)
- The three `packages/*` are built with **Rollup** and must be built before client or server tests run

## Commands

```bash
# Install
pnpm install

# Dev (builds packages, then concurrently runs server + client + package watchers)
pnpm dev

# Build packages first — REQUIRED before running client/server tests
pnpm packages:build

# Lint all workspaces
pnpm lint

# Test
pnpm server:test                          # Mocha — runs server/test/lifecycle.test.js + integration/**/*.test.js + utils/**/*.test.js
pnpm client:test                          # Jest — client/src/**/__tests__/** and *.{spec,test}.{js,jsx}
pnpm test:e2e                             # Playwright — from tests/ dir (requires running server on :3000)

# Run a single server test file (Mocha)
cd server && pnpm exec mocha test/lifecycle.test.js test/integration/models/User.test.js

# Run a single client test file (Jest)
cd client && node scripts/test.js --testPathPattern="src/tests/i18n"

# DB management (run from root or with -C server)
pnpm server:db:migrate
pnpm server:db:rollback_one
pnpm server:db:seed
```

## Critical: packages must be built before tests

`pnpm packages:build` (or `pnpm pretest` automatically) is required because `client` and `server` import from `@4gaboards/enums`, `@4gaboards/locales`, `@4gaboards/utils` via workspace `dist/` — not source.

## Code Style

- **Prettier**: `printWidth: 210` (very long lines allowed), `singleQuote: true`, `trailingComma: 'all'`; locales files use `printWidth: 4000`
- **JSX only in `.jsx` files** — `react/jsx-filename-extension` enforces this; `.js` files must not contain JSX
- **Import order** (enforced by `eslint-plugin-perfectionist`): `react` → `builtin/external/internal` → `parent/sibling/index` → `css` → `module.scss`; one blank line between groups
- **No `for...of` or `for...in`** loops — `no-restricted-syntax` bans both; use array methods (`map`, `filter`, `reduce`, `Object.keys/values/entries`) instead
- **No prop spreading** on JSX components — `react/jsx-props-no-spreading` is enforced (use explicit spread with `explicitSpread: 'ignore'` if needed)
- **PropTypes required** for all React props (`react/prop-types: error`); `any`, `array`, `object` propTypes are forbidden — use `arrayOf`/`shape` instead
- **Modules with single export must use default export** — `import-x/prefer-default-export: error`
- **No circular imports** — `import-x/no-cycle` with `maxDepth: ∞`
- **Packages can't import each other via relative paths** — `import-x/no-relative-packages: error`; use workspace name (`@4gaboards/utils`)
- Server uses **CommonJS** (`require`); client uses **ESM** (`import`)

## Server Architecture

- Sails.js with blueprints, grunt, i18n, and session hooks **disabled** (see [`server/.sailsrc`](server/.sailsrc))
- Server tests require a live Sails instance — `server/test/lifecycle.test.js` lifts/lowers sails in `before`/`after` hooks; it must always be first in the mocha run order
- Environment variables loaded via `dotenv` from `server/.env` (copy from [`server/.env.sample`](server/.env.sample)); `DATABASE_URL`, `BASE_URL`, and `SECRET_KEY` are required

## Testing Notes

- **Server tests** use **Mocha + Chai** (`expect` style); models are accessed as globals (Sails globals, no `require`)
- **Client tests** use **Jest + jsdom + @testing-library/react**; CSS modules are mocked via `identity-obj-proxy`; module CSS (`*.module.scss`) imports are mocked
- **E2E tests** use **Playwright** from `tests/` directory; require a running server on `http://localhost:3000`
- `pnpm ci:test:e2e` starts the server automatically before running Playwright
