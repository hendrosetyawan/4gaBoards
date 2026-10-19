# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Non-Obvious Coding Rules

- **Build packages before editing client/server** — `pnpm packages:build` regenerates `packages/*/dist/`; client and server both import from `dist/`, not source. Stale `dist/` will cause silent wrong-behavior bugs.
- **JSX only in `.jsx` — never `.js`** — ESLint will error if you add JSX to a `.js` file. Always create React components with `.jsx` extension.
- **No `for...of` / `for...in`** — banned by `no-restricted-syntax`. Use `.map()`, `.filter()`, `.reduce()`, `Object.keys/values/entries()` instead — even in `for` loop equivalents.
- **No prop spreading on custom components** — `react/jsx-props-no-spreading` is `error`. Always list props explicitly; the only exception is `explicitSpread: 'ignore'` (spreading a named object).
- **PropTypes: no `any`, `array`, `object`** — use `PropTypes.arrayOf(...)` and `PropTypes.shape({...})` instead. Missing propTypes is also an error.
- **All props need `defaultProps` unless required** — `react/require-default-props: error` with `forbidDefaultForRequired: true`.
- **Single-export modules must use default export** — `import-x/prefer-default-export: error`. A file exporting only one thing must use `export default`.
- **No `++`/`--`** outside for-loop afterthoughts (`no-plusplus: error`). Use `+= 1` instead.
- **Import order is enforced** — Groups: `react` → external/builtin/internal → parent/sibling/index → plain CSS → `*.module.scss`. One blank line between groups. Violations are `warn` but the linter will report them.
- **Server uses `global-require`** and CJS; all server `require()` calls must be at the top level (not inside functions) — `global-require: error`. The Sails.js globals (`sails`, model names like `User`) are available without require in the server context.
- **Workspace cross-package imports must use package names** — never use relative paths across packages (e.g. `../../packages/utils`). Use `@4gaboards/utils`, `@4gaboards/enums`, `@4gaboards/locales`.
- **Server tests need Sails lifted** — always keep `server/test/lifecycle.test.js` as the first file in mocha runs; it provides `before`/`after` that lift and lower sails. Models are sails globals in tests (no import needed).
