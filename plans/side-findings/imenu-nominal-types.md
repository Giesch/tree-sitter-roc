# S02: Imenu lists nominal types

Status: done (4ca916aaf5ada32dd1c77f7ad42028a5015c19c0)

Verification: 29/29 ERT tests passed, with zero unexpected results and zero skips, against grammar pin `2026-10-13.01`. Pre-existing test failures were repaired separately in `51816b750a11acef107f72c51f068795dde67b87` (28/28 tests passed before the imenu change).

## Goal

`M-x imenu` in a Roc buffer lists nominal type definitions (`Name := ...`).

## Depends on

None. This change works with the current grammar pin (`2026-10-13.01`), which has `nominal_type_def`.

## Context

- Repository: `~/Projects/roc-ts-mode`. Load the `roc-ts-mode-maintenance` skill first.
- `roc-ts--ts-simple-imenu-settings` (`roc-ts-mode.el:581-585`):
  ```elisp
  `(("Definition" ,(rx bos "value_declaration" eos) nil nil)
    ("Type alias" ,(rx bos "alias_type_def" eos) nil nil)
    ("Opaque type" ,(rx bos "opaque_type_def" eos) nil nil))
  ```
- `roc-ts--ts-defun-name` (`:595-605`) handles any node type that contains `type_def`, so it already returns names for `nominal_type_def`.
- Tests: `tests/roc-ts-mode-test.el`.

## Steps

1. Add `("Nominal type" ,(rx bos "nominal_type_def" eos) nil nil)` to the settings.
2. Add an ERT test. Insert `Color := [Red, Green]` and `x = 1` into a temp buffer, enable `roc-ts-mode`, call `(treesit-simple-imenu)`, and assert that the "Nominal type" entry contains `Color`. Do not add a `skip-unless`. The tests assume the pinned grammar is installed.
3. Check parens. Load the source file in a disposable daemon, verify `symbol-file`, and run all ERT tests.
4. Commit in the mode repository. Publish only when the user asks, through the maintenance skill.

## Out of scope

- Other imenu changes. `emacs-mode/03` adds `var_declaration` later.

## Verification

- New ERT test passes. Zero unexpected results in the full suite.

## Done when

- [x] Settings list `nominal_type_def`.
- [x] ERT test passes.
