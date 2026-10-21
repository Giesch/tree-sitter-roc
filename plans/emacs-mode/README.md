# Emacs-mode plans

## Current checkpoint: E03 locally committed (2026-10-21)

E03 is complete and locally committed at `7558c81f94e3dcd959d7ddce95892e8b5019a15c`, after mode commit `f258f2f` on `emacs-grammar-2026-10-20`. The mode working tree is clean. Runtime tests were not re-run during the commit-only step; E04 requires a fresh isolated run before publication. Field-based declaration lookup, mutable declaration navigation/imenu, and sibling-annotation indentation are verified: full ERT 31/31, zero unexpected/skipped, strict compilation and Lisp checks passed. See [E03 execution evidence](03-structural-elisp.md#execution-evidence-2026-10-21-local-unpublished). E04 remains pending; nothing was published or installed into the daily editor. The E01/E02 checkpoint descriptions below are historical; their statements that E03 is pending are superseded by this checkpoint.

All work is in `~/Projects/roc-ts-mode`. Load the `roc-ts-mode-maintenance` skill before each plan. It defines the source checkout, test procedure, publish steps, and Doom pin. Use the `emacsclient` and `testing-emacs-daemons` skills for all Emacs evaluation.

## Goal

roc-ts-mode uses the pinned grammar and generated face-only Emacs queries from tree-sitter-roc. The mode keeps one local font-lock callback for derived associated-method names because the current Emacs 30.1/libtree-sitter 0.26 runtime cannot evaluate the required query predicates; indentation, imenu, defun navigation, hideshow, and commands remain in Elisp.

## Compatibility policy

The mode supports exactly the grammar tag it pins. It does not support older or newer grammars.
- Do not add runtime probes, fallback rules, or version checks for grammar features.
- Do not add test skips that depend on grammar features. Tests assume the pinned grammar is installed.
- Do not add a startup check that warns about a different installed grammar.
- [../side-findings/remove-grammar-compat.md](../side-findings/remove-grammar-compat.md) removes the existing compatibility code.

## Constraint

The hand-written font-lock rules name nodes that the grammar plans remove (`suffix_operator`, `(match)`, `(where)`, `(as)`, `(exposing)`, `expr_body` positions). The pin, the font-lock source, and the structural Elisp must change in the same published commit. Do plans 01 to 03 on one branch. Publish only in plan 04. Intermediate commits on the branch can fail tests; the branch must pass before plan 04.

## E01 complete; mode remains unpublished

The grammar prerequisite is published on fork `master` and tag `2026-10-20.01`, both at tree-sitter-roc `e13e92b3ee585a00ac34c00372b9f8233e933cad`. Both `npm run ci` and `just ci` passed in a clean committed-source worktree using existing project-local dependencies. A fresh `npm ci --offline` failed compiling a native dependency under Node 24, so fresh dependency installation was not verified. E01 installed the actual remote tag with `treesit-install-language-grammar` into a fresh disposable daemon's private destination; it reported Roc available, `try_operator` in `f = |x| x?`, zero `ERROR` nodes in `all_syntax_test.roc`, and focused S06 plus imenu ERT 2/2 with zero unexpected/skipped. The private daemon stopped and its isolated test root was cleaned after checks.

At the E01 checkpoint, source branch `emacs-grammar-2026-10-20` pinned `2026-10-20.01` and included the Q02 callback adapter and S06 after-header rule/test in local commit `5eb4f8fb6b5b9c737b47b4894351e781a78654a6` (message-only amend of `547a56d`). The published mode and daily editor remained on `2026-10-13.01`; the grammar push did not authorize a mode or daily-editor update. Both the earlier local-library check and the later actual remote-tag installer check passed focused S06 and neighboring imenu ERT (2/2, zero unexpected/skipped in each). A targeted font-lock ERT failed on the old `(where)` query. E02 subsequently replaced the stale font-lock rules and passed the full 29-test suite; E03 structural migration and E04 publication remain separate gates. See [E01 execution notes](01-grammar-tag-and-pin.md#execution-notes-grammar-published-mode-unpublished) and [S06](../side-findings/module-header-indentation.md#execution-evidence-local-unpublished).

## Plans

| Plan | Change | Depends on |
|---|---|---|
| [01-grammar-tag-and-pin.md](01-grammar-tag-and-pin.md) | Tag the grammar, update the pin, install it in a test daemon | all grammar plans, query plans 01 and 02 |
| [02-consume-generated-queries.md](02-consume-generated-queries.md) | Generated face-only queries plus one mode-local derived-name callback rule | 01; new query-contract tag |
| [03-structural-elisp.md](03-structural-elisp.md) | Defun name, imenu, indent rules use fields and sibling annotations | 01 |
| [04-publish.md](04-publish.md) | Push, Doom pin, refresh open buffers | 02, 03 |

E02 is locally committed at roc-ts-mode `f258f2f674541271f998641686441d0c1b0cf803` and not published. Its face-only query contract is tree-sitter-roc `6d85644f993edc543be0214bfffe60905d8e66d1`, separately approved and published on fork `master` and tag `2026-10-20.02`; both remote refs resolve to that commit. The mode pins `.02`, embeds the tagged generated queries, and retains one final mode-local derived-name callback rule. A fresh private tag install, strict warning-free compilation of four edited Lisp files using actual sibling autoloads without suppressed warnings, full ERT 29/29 (zero unexpected/skipped), 47 ordered settings, and deterministic source-verified generation passed. Old installed mode revision `59c40a38` with grammar `.01` was captured privately after the old `/tmp` reference disappeared; inspected PNGs and snapshots support matching layout and intended highlighting changes at 110 columns. Screenshot navigation did **not** execute and is not counted as a pass; it lies outside E02's changed highlighting behavior. See [E02](02-consume-generated-queries.md) for provenance, artifact paths, and cleanup. E03 subsequently completed the structural work in local commit `7558c81`; E04 alone gates mode publication, Doom pin, and daily-editor changes.
