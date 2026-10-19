# Bob Fixes the Board: fixing 4ga Boards #279 with IBM Bob

**Author:** Hendro Setyawan, Texas A&M University

A personal project to learn **IBM Bob**, IBM's AI coding assistant. I started it for AngelHack × IBM's *Building with IBM Bob* (October 2026) but missed the submission deadline, so I finished it for fun. It follows the event's student theme: take an unfamiliar application, use Bob to understand it, and fix a real issue.

This fork of [4ga Boards](https://github.com/RARgames/4gaBoards), an open-source Trello-style kanban app, holds the work: a fix for a real open bug, with tests, made with Bob.

## The issue

[#279](https://github.com/RARgames/4gaBoards/issues/279) (duplicate [#159](https://github.com/RARgames/4gaBoards/issues/159)): open a card, then delete the list that contains it. The whole board is replaced by **"Card Not Found"**, and the URL stays on the deleted card.

## What IBM Bob did

All of the following was done by Bob Shell 2.0.6. The session records are in [`docs/bob/`](docs/bob/).

| Step | What Bob did |
| --- | --- |
| `/init` | Analyzed the monorepo and wrote [`AGENTS.md`](AGENTS.md) and per-mode rules in [`.bob/`](.bob/) (stack, commands, code style). 2.68 Bobcoins. |
| Root cause | Compared `deleteCard` / `handleCardDelete` in `cards.js` (which call `goToBoard` before removing an open card) with `deleteList` / `handleListDelete` in `lists.js` (which never check the open card). |
| Plan | Found `selectors.selectCardById` to read the open card's `listId` before the list is removed; covered the local delete and the real-time socket handler. |
| Implement | Created branch `fix/delete-list-open-card-279` and added the check to both sagas in `client/src/sagas/core/services/lists.js` (+21 lines). |
| Test | Wrote `client/src/sagas/core/services/lists.test.js` (6 tests), fixed its own test-setup and lint errors, and finished at 6/6 passing with 0 ESLint warnings. |
| Review & commit | Reviewed both files against `AGENTS.md` (all checks passed), committed only those two files as [`44736daa`](https://github.com/hendrosetyawan/4gaBoards/commit/44736daab096e861c755edfa925e0e586d98b1a3), pushed the branch, and drafted a PR description. 0.40 Bobcoins. |

The fix run was Bob task `a44a857ea1db5b46d38622f7a498b40f`: 47 tool calls in about 3 minutes, 3.08 Bobcoins. In total, Bob used about **6.2 Bobcoins**.

The fix itself is on branch [`fix/delete-list-open-card-279`](https://github.com/hendrosetyawan/4gaBoards/tree/fix/delete-list-open-card-279). This `ibm-bob-project` branch adds this write-up, Bob's session records in [`docs/bob/`](docs/bob/), and Bob's `/init` output.

## Verification

- `lists.test.js`: **6/6 pass** with the fix, **6/6 fail** without it.
- ESLint on both files: 0 errors, 0 warnings.
- Manual, in the running app:
  1. Open a card, delete its list → back on the board (no "Card Not Found").
  2. Open a card in tab A, delete its list from tab B → tab A returns to the board by itself (socket path).

## Run it locally

Requires Node 24, pnpm 12 and PostgreSQL.

```bash
pnpm i
cp server/.env.sample server/.env   # set DATABASE_URL to your database
pnpm server:db:init
pnpm dev                            # http://localhost:3000, log in with demo / demo
```

Reproduce: open board **Learn 4ga Boards**, click a card, then on that card's list choose **Edit List → Delete List → Delete list**. You stay on the board.

Run the test:

```bash
pnpm packages:build
cd client && node scripts/test.js --watchAll=false src/sagas/core/services/lists.test.js
```

## Next steps

- Open the fix as a pull request to [RARgames/4gaBoards](https://github.com/RARgames/4gaBoards).
- Fix [#491](https://github.com/RARgames/4gaBoards/issues/491) (task created in the wrong card when switching cards).
- Add "Export board to CSV".
