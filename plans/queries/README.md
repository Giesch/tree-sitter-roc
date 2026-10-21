# Query plans

## Current state — 2026-10-21

Q01 and Q02 are complete and published, with the final face-only Emacs query contract in grammar tag `2026-10-20.02`. E04 delivered their mode integration to the daily editor. Q03 is complete: five topic fixtures cover the base captures, the deliberate wrong-capture check failed as expected, and `just ci` passed. Leaf plans record the approved Q02 output adjustment and historical contract changes.

## Goal

`queries/*.scm` is the one source of highlighting rules for every editor, including Emacs. Each editor keeps its own capture names through its own map, and an editor can still ship hand-written queries.

## Architecture

```
queries/*.scm                 base queries, Helix capture names, source of truth
query-maps/<editor>.json      base capture -> editor capture
tools/template-queries.js     applies each map
queries-generated/<editor>/   generated output, checked by `just queries-check`
```

Rules:
- A base capture describes meaning, not a face. Example: `@keyword.control.exception` for `crash`.
- A distinction that one editor wants and others do not uses the `.roc-special.<name>` suffix. Example: `@keyword.control.return.roc-special.try` for `?`. Every map must list every capture (the generator fails on a missing entry), so each editor decides: map it to its own name, map it to the unsuffixed name, or map it to `""` to drop it.
- Later patterns win. Keep lower-priority patterns first.
- An editor that wants rules the base cannot express keeps them in its own repository, after the generated rules. Neovim keeps `neovim/` as it is.

## Plans

| Plan | Change | Depends on |
|---|---|---|
| [01-shared-query-parity.md](01-shared-query-parity.md) | Base queries cover everything roc-ts-mode highlights | all grammar plans |
| [02-emacs-query-map.md](02-emacs-query-map.md) | Emacs map, feature sections, predicate check in the generator | 01 |
| [03-highlight-tests.md](03-highlight-tests.md) | `test/highlight/` assertions for the base captures | 01 |

Do plans 01 and 02 after all grammar plans and before the Emacs-mode plans. The Emacs-mode plans use the output of both.

Plan 03 is independent of plan 02 and of the Emacs-mode plans. Do it at any point after plan 01.
