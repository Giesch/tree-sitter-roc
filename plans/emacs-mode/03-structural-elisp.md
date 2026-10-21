# E03: Structural Elisp uses fields and sibling annotations

Status: done (roc-ts-mode `7558c81f94e3dcd959d7ddce95892e8b5019a15c`); locally committed, not published.

## Goal

Defun navigation, imenu, and indentation use named fields and the sibling-annotation shape from the new grammar.

## Depends on

[01-grammar-tag-and-pin.md](01-grammar-tag-and-pin.md).

## Context

- `roc-ts--ts-defun-name` (`roc-ts-mode.el:595-605`) uses nested `treesit-node-child ... 0` calls.
- `roc-ts--ts-defun-p` (`:587-593`) matches `value_declaration`, any `*type_def`, and children of `file`.
- `roc-ts--ts-simple-imenu-settings` (`:581-585`) lists `value_declaration`, `alias_type_def`, `opaque_type_def`. The side-finding plan [../side-findings/imenu-nominal-types.md](../side-findings/imenu-nominal-types.md) adds `nominal_type_def`. If that plan is merged, keep its entry.
- Indent rule `(n-p-gp "annotation_type_def" "value_declaration" nil)` (`:467`) handles annotations inside a block declaration. After grammar plan 04, an annotation is never a child of `value_declaration`.
- New grammar fields (grammar plan 03):
  - `value_declaration` has `pattern:` on `decl_left`.
  - `annotation_type_def` and `inferred_associated_annotation` have `name:` on `annotation_pre_colon`.
  - `alias_type_def`, `opaque_type_def`, `nominal_type_def` have `name:` on `apply_type`.
  - `var_declaration` has `name:`.
- Indent tests: `tests/roc-ts-mode-examples.erts`, `tests/roc-ts-mode-newline-and-indent.erts`.

## Steps

1. Rewrite `roc-ts--ts-defun-name` with `treesit-node-child-by-field-name`:
   - `value_declaration`: `pattern` → if the child is `identifier_pattern`, return its identifier text; otherwise return the `decl_left` text.
   - `var_declaration`: `name` text.
   - `*type_def`: `name` → `apply_type` → `concrete_type` text.
   - `annotation_type_def`: `name` text. Include annotations only if step 3 adds them to imenu.
2. Add `var_declaration` to `roc-ts--ts-defun-p`.
3. Update imenu settings: add `("Variable" "\\`var_declaration\\'" nil nil)`. Keep `nominal_type_def` from the side-finding plan, or add it as `("Nominal type" ...)` if it is absent.
4. Delete the indent rule at `:467`. Check whether an annotation inside a block needs a rule now: run the indent ERT tests. If a block annotation indents wrong, add `((n-p-gp "annotation_type_def" "body_expression" nil) parent-bol roc-ts--indent-offset)` or the form that passes.
5. Add an ERT test for `roc-ts--ts-defun-name` on: `f = |x| x`, `(a, b) = t`, `var n = 0`, `T : [A]`, `N := [B]`.
6. Add an erts case for a block with an annotation followed by its declaration.
7. Check parens, byte-compile, and run all ERT tests in the disposable daemon.

## Out of scope

- Font-lock (plan 02).
- Publishing (plan 04).

## Verification

- `grep -n 'treesit-node-child [^b]' roc-ts-mode.el` shows no first-child chains in name lookup.
- ERT: zero unexpected results.
- `M-x imenu` in `tests/all_syntax_test.roc` lists definitions, type aliases, opaque types, nominal types, and variables.

## Done when

- [x] Name lookup enters declarations through fields; unfielded children are selected by node type, never positional child chains.
- [x] Obsolete annotation-under-declaration indent rule is gone, and indent tests pass without a replacement rule.
- [x] New ERT and erts cases pass.

## Execution evidence (2026-10-21; local, unpublished)

Implemented in `/home/danknutson/Projects/roc-ts-mode`, on branch `emacs-grammar-2026-10-20`, and locally committed at `7558c81f94e3dcd959d7ddce95892e8b5019a15c` after E02 commit `f258f2f674541271f998641686441d0c1b0cf803`. The commit contains only `roc-ts-mode.el`, `tests/roc-ts-mode-test.el`, and `tests/roc-ts-mode-examples.erts`. The staged patch passed `git diff --cached --check`; the mode working tree was clean afterward. Runtime verification below predates this commit and was not re-run during the commit-only step.

- `roc-ts--ts-defun-name` reads `pattern` or `name`. The grammar does not field the inner `identifier_pattern` or `concrete_type` children, so these are selected by type. Tuple patterns retain their complete text; generic type names exclude arguments. Annotations remain excluded from imenu and return nil from name lookup.
- Mutable declarations participate in defun selection and the new Variable imenu category. Existing nominal-type support remains intact.
- Tests cover simple and tuple bindings, mutable declarations, aliases, generic aliases, nominal and opaque types, generic opaque types, annotation exclusion, imenu categories, and nested mutable-declaration selection. A new erts fixture verifies sibling annotation/declaration indentation inside a block.
- Plain Emacs 30.1 disposable daemon: PID `892153`, socket `/tmp/emacs-test.mYm7jF3u/server`. Installed the mode's pinned remote grammar `2026-10-20.02` into its private home; verified Roc availability and exact source provenance. No site-start library was loaded.
- Lisp checker passed for both edited `.el` files. Strict byte-compilation with `byte-compile-error-on-warn` passed for both files, with outputs under the private root. Initial compilation lacked the sibling `roc-ts-start` definitions; loading the package's sibling command files resolved it without suppressing warnings.
- Full ERT: **31 total, 31 completed, zero unexpected, zero skipped**, including the formatter comparison and both erts runners. Earlier new fixtures used invalid opaque/generic syntax; corrected to `::` and parenthesized type arguments after inspecting actual trees. An intermediate failed-assertion reporting run timed out and was interrupted with SIGUSR2 against the verified owned PID; the final full run completed normally.
- Programmatic `treesit-simple-imenu` on `tests/all_syntax_test.roc`: Definition 35, Variable 4, Type alias 1, Opaque type 1, Nominal type 1; no parse ERROR. This verifies the index, not an interactive graphical menu.
- `git diff --check` passed. No public API, dependency, or grammar contract changed; no AGENTS.md update was needed for this internal migration.
- Cleanup verified: owned daemon exited, socket refused connections, and `/tmp/emacs-test.mYm7jF3u` was removed.
- E04 remains the publication gate: E03 is committed locally, but no push, Doom pin change, installed-package update, or daily-editor reload was performed.
