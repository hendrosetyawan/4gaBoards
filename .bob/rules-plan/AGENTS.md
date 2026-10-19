# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Non-Obvious Architectural Constraints

- **`packages/*` must be built before client or server start/test** — `@4gaboards/enums`, `@4gaboards/locales`, `@4gaboards/utils` ship CJS + ESM from `dist/`. Running without `pnpm packages:build` will fail silently with stale or missing exports.
- **Dual module format in packages** — Each package exports both CJS (`dist/cjs/`) for the server and ESM (`dist/esm/`) for the client. Rollup config generates both. Type definitions in `dist/index.d.ts`.
- **Sails + Knex dual data layer** — Sails ORM handles model CRUD and associations (via `sails-postgresql`). Knex (`server/db/`) handles schema migrations only. These are not interchangeable — do not add migration logic to models.
- **Client state: Redux + Redux-ORM + Redux-Saga** — The client uses `redux-orm` for relational model state (not a standard Redux pattern) and `redux-saga` for side-effects. New features must integrate with existing ORM models in `client/src/models/` and sagas in `client/src/sagas/`.
- **Server is a single-page-app backend** — Sails serves the built React app from `server/public/`. In development the client webpack-dev-server runs on a separate port with proxy to Sails on :1337.
- **Socket.IO real-time** — `sails-hook-sockets` provides WebSocket support. Client uses `socket.io-client`. Actions that mutate state are broadcast to connected clients via sails socket rooms.
- **No circular dependencies enforced** — `import-x/no-cycle` with `maxDepth: ∞` is a hard error. Keep the dependency graph acyclic across all packages and within client/server.
- **Locales package has a `printWidth: 4000` Prettier override** — locale files are intentionally one-liners to enable easy git diffs; do not reformat them with default Prettier settings.
