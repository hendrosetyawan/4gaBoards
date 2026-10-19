# IBM Bob session: fix for issue #279

Captured from the Bob Shell 2.0.6 terminal output of a headless run (`bob run`, Agent mode).
Task ID `a44a857ea1db5b46d38622f7a498b40f` · 2026-10-19 04:28–04:31 UTC.
Tool-call arguments are shortened the same way Bob's terminal shows them.

## Prompt

> Fix GitHub issue 279 in this repo (duplicate of 159). Bug: open a card, then delete the list that contains it; the app shows Card Not Found and the URL stays on /cards/<id>. Deleting the card itself works and returns to the board. Work in this order and show each step: 1) Explain the root cause by comparing @client/src/sagas/core/services/lists.js (deleteList, handleListDelete) with @client/src/sagas/core/services/cards.js (deleteCard, handleCardDelete), citing exact lines. 2) Write a short plan for a minimal fix: if the currently open card (selectors.selectPath gives cardId and boardId) belongs to the list being deleted, call goToBoard(boardId) before the list is removed; find the selector that returns a card with its listId; cover both the local delete and the socket handler. 3) Create git branch fix/delete-list-open-card-279 and implement it, following the AGENTS.md style rules. 4) Add a Jest test next to the existing client tests. 5) Run that test with pnpm and run eslint on the changed files, and fix any failures. Node 24 and pnpm 12 are already on PATH. Do not commit. Do not change unrelated files.

## Exploration (tool calls 1–19)

Bob read `cards.js` and `lists.js`, the selectors (`cards.js`, `router.js`, `lists.js`, `index.js`), `sagas/core/services/router.js`, the existing tests (`tests/i18n.test.js`, `utils/local-id.test.js`) and `client/package.json` to learn the Jest setup.

## Step 1: Root cause (Bob's answer)

> `deleteList` (lists.js line 74) and `handleListDelete` (line 88) dispatch the remove action immediately. Neither checks whether the currently open card lives in the list being deleted.
>
> `deleteCard` (cards.js lines 191–194) and `handleCardDelete` (lines 217–220) read `cardId`/`boardId` from `selectPath` and call `goToBoard(boardId)` **before** the Redux action removes the card.
>
> When you delete a list while viewing one of its cards: `actions.deleteList(id)` makes the ORM remove the list and its cards; `selectPath` then finds no card, the router still sees `/cards/<id>`, and the app renders "Card Not Found" with the stale URL.

## Step 2: Plan (Bob's answer)

> Use `selectors.selectCardById` (selectors/cards.js line 31), which returns the card including `.listId`. In `deleteList(id)`: read `{ cardId, boardId }` from `selectPath`; if `cardId` is set, select the card; if `card.listId === id`, call `goToBoard(boardId)` before dispatching. Apply the same pattern in `handleListDelete(list)` (socket push). Import `goToBoard` from `./router`, as `cards.js` already does.

## Step 3: Implementation

- `git checkout -b fix/delete-list-open-card-279`
- `apply_diff` on `client/src/sagas/core/services/lists.js`: added the import and the check to both sagas (+21 lines).

## Step 4: Tests

- Checked for `@redux-saga/testing-utils` and `redux-saga-test-plan` (neither installed), so Bob stepped through the generators with `redux-saga/effects` descriptors.
- Wrote `client/src/sagas/core/services/lists.test.js`.

## Step 5: Validation, and Bob fixing its own failures

1. `pnpm packages:build`, then ran the test: **failed**, because importing the real selectors loads `Config.js`, which reads `window.BASE_URL`.
2. Bob rewrote the test to `jest.mock` the selectors, actions, API, request and router modules, so no ORM or DOM wiring is needed. **6/6 passed.**
3. `pnpm lint:js` on both files: **errors**, because the shared ESLint config doesn't declare the `jest` global. Bob added `/* global jest */`.
4. Lint again: only import-order warnings left. Bob fixed the order.
5. Lint again: **0 errors, 0 warnings.** Test again: **6/6 pass.**

## Bob's summary

> **Step 3 — Implementation:** `lists.js`: added `import { goToBoard } from './router'`; `deleteList` reads the path, selects the card, and calls `goToBoard` if `card.listId === id`; `handleListDelete` uses the same pattern with `list.id`.
> **Step 4 — Tests:** `lists.test.js`: 6 generator-stepping tests; heavy dependencies are mocked.
> **Step 5 — Validation:** `pnpm lint:js` → 0 errors, 0 warnings on both files. Jest `lists.test.js` → 6/6 pass.

```
Task Summary
Total Cost:              3.08
Total Duration:          3m 4s
Assistant Messages:      22
Tool Calls:              47
Task ID:                 a44a857ea1db5b46d38622f7a498b40f
```

## Earlier: `/init`

Before the fix, an interactive Bob session ran `/init`, which wrote `AGENTS.md` and `.bob/rules-{agent,ask,plan}/AGENTS.md` (2.68 Bobcoins).
