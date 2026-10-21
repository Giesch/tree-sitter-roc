# S03: Remove grammar-version compatibility from roc-ts-mode

Status: done (f17d62f8aa9f92fd0d767226d34d176d89ffa3ec)

Verification: 28/28 ERT tests passed with zero unexpected results and zero skips against grammar pin `2026-10-13.01`, independently reproduced by the alternate-family reviewer. The planned keyword-test merge reduced the suite from 29 tests to 28.

## Goal

roc-ts-mode supports only the grammar tag it pins. The mode has no runtime probes for grammar features, and the tests do not skip on grammar features.

## Depends on

None. This change works with the current grammar pin (`2026-10-13.01`), which has `while_expr`, `break_expr`, `crash_expr`, and `expect`.

## Context

- Repository: `~/Projects/roc-ts-mode`. Load the `roc-ts-mode-maintenance` skill first.
- Policy: the mode works with the exact tag in `treesit-language-source-alist` (`roc-ts-mode.el:149-151`). Older and newer grammars are not supported.
- Compatibility code in `roc-ts-mode.el`:
  - `roc-ts--ts-conditional-keyword-patterns` (`:423-428`): `"while"`, `(break_expr)`, `"crash"`.
  - `roc-ts--ts-conditional-keyword-rules` (`:430-448`): compiles each pattern with `ignore-errors` and drops the ones that fail. It gives `"crash"` the face `roc-ts-crash-face`.
  - `roc-ts--ts-setup` (`:612-615`) appends the conditional rules to `roc-ts--ts-font-lock-rules`.
- Compatibility skips in `tests/roc-ts-mode-test.el`:
  - `roc-ts-font-lock-conditional-keywords` (`:220-229`): `skip-unless` on the conditional rules. Its docstring describes older grammars.
  - `roc-ts-font-lock-crash` (`:231-240`): `skip-unless` on the conditional rules.
  - `roc-ts-font-lock-expect-statement` (`:242-249`): `skip-unless` on compiling `(expect)`. Its docstring describes older grammars.
  - `roc-ts-block-statements-with-updated-grammar` (`:251-275`): `skip-unless` on compiling `(expect)`.
- Keep `(when (treesit-ready-p 'roc) ...)` in the mode body (`roc-ts-mode.el:140`). It handles a missing grammar, not a different grammar version.

## Steps

1. Add the three patterns to the `keywords` feature in `roc-ts--ts-font-lock-rules`: `"while" @font-lock-keyword-face`, `(break_expr) @font-lock-keyword-face`, `"crash" @roc-ts-crash-face`. Keep the rule order: these patterns go in the existing `keywords` block.
2. Delete `roc-ts--ts-conditional-keyword-patterns` and `roc-ts--ts-conditional-keyword-rules`.
3. In `roc-ts--ts-setup`, pass `roc-ts--ts-font-lock-rules` to `treesit-font-lock-rules` without `append`.
4. In the tests:
   - Delete the four `skip-unless` forms.
   - Merge the `while` and `break` needles from `roc-ts-font-lock-conditional-keywords` into `roc-ts-font-lock-keywords`, and delete the conditional test.
   - Rewrite docstrings that mention older or newer grammars. State only what the test checks.
   - Rename `roc-ts-block-statements-with-updated-grammar` to `roc-ts-block-statements`.
5. `grep -n 'ignore-errors' roc-ts-mode.el`. The two remaining uses in `roc-ts--indent-line` (`:543`, `:559`) guard line navigation, not grammar features. Keep them.
6. Check parens. Load the source file in a disposable daemon with the pinned grammar, verify `symbol-file`, and run all ERT tests. The skipped count must be zero for the font-lock tests.
7. Commit in the mode repository. Publish only when the user asks, through the maintenance skill.

## Out of scope

- A startup check that warns about a mismatched grammar. The policy does not require it.
- Font-lock changes beyond moving the three patterns.

## Verification

- `grep -n 'conditional\|skip-unless\|treesit-query-compile' roc-ts-mode.el tests/roc-ts-mode-test.el` returns nothing, except a `skip-unless` that a later plan adds for a non-grammar reason.
- ERT: zero unexpected results and zero skipped font-lock tests.

## Done when

- [x] No grammar probes in the mode.
- [x] No grammar-feature skips in the tests.
- [x] All ERT tests pass with the pinned grammar.
