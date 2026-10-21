# E01: Grammar tag and mode pin

Status: done (grammar `e13e92b3ee585a00ac34c00372b9f8233e933cad`, published tag `2026-10-20.01`; local mode `5eb4f8fb6b5b9c737b47b4894351e781a78654a6`). E02/E03 and mode publication remain pending.

## Goal

A grammar tag contains all grammar plans and query plans 01 and 02. The mode source pins that tag. Before publication, a disposable Emacs daemon verifies a local build from the tagged commit; after the approval-gated push, a new disposable daemon must verify an actual installation from the remote tag for plans 02 and 03.

## Depends on

All plans in [../grammar/](../grammar/README.md), [../queries/01-shared-query-parity.md](../queries/01-shared-query-parity.md), and [../queries/02-emacs-query-map.md](../queries/02-emacs-query-map.md), committed in tree-sitter-roc. [../queries/03-highlight-tests.md](../queries/03-highlight-tests.md) is not required.

## Context

- The unpublished mode source branch pins published grammar tag `2026-10-20.01` in `roc-ts-mode.el:149-151`; the published mode and daily installation still use `2026-10-13.01`. Do not load the new source with the old installed grammar.
- Tag scheme in tree-sitter-roc: `YYYY-MM-DD.NN`, for example `2026-10-13.01`. Remote name for the fork: `fork` (`git@github.com:Giesch/tree-sitter-roc.git`). Upstream is `upstream`; do not push there.
- `treesit-install-language-grammar` clones the tag and compiles `src/parser.c` and `src/scanner.c`. The tag must contain regenerated parser files.

## Steps

1. In tree-sitter-roc, confirm `just ci` passes on the commit to tag, and `git status` is clean.
2. Ask the user before pushing. Pushing a tag is outward-facing.
3. Create the tag with today's date and the next free `NN`: `git tag <tag>`. Push: `git push fork <branch> <tag>`.
4. In roc-ts-mode, create a branch. Update the pin string to the new tag. Include [../side-findings/module-header-indentation.md](../side-findings/module-header-indentation.md) in the same mode pin-update commit: the module-header indentation rule must use `module_exposes`, with focused regression coverage. Coordinate test execution with E02 if stale font-lock queries prevent initialization; do not add a compatibility fallback.
5. Start a disposable daemon. Set `treesit-extra-load-path` to a private directory so the install cannot replace the daily grammar. Before push, a library built from the exact local tagged worktree can verify parser behavior, but it cannot establish that the remote tag installs. After authorized push, install the published tag with `treesit-install-language-grammar` into a new private directory and verify its provenance.
6. In that daemon, confirm `(treesit-language-available-p 'roc)` and parse `tests/all_syntax_test.roc` with no new `ERROR` nodes. Compare with the old grammar if `ERROR` nodes appear; it is a reference only, not a supported fallback. Inspect the `f = |x| x?` tree for `try_operator` using a buffer parser on Emacs 30.1, where `treesit-parse-string` is unavailable.
7. E01 may run a targeted font-lock test to identify an intermediate blocker. Do not treat that failure as a completed E02 migration or claim a passing full suite until E02/E03 are done.

## Execution notes (grammar published; mode unpublished)

- Grammar prerequisite: tree-sitter-roc `master` commit `e13e92b3ee585a00ac34c00372b9f8233e933cad`; tag `2026-10-20.01` and fork default branch `master` now both resolve remotely to that exact hash. Before the non-force push, fork `master` was `8146ec4a598c80ea3628098e72fd0e16c416241d`, an ancestor of the intended release; the fork tag did not exist. A separate detached worktree at `/tmp/roc-grammar-release-e01` was clean at the release commit. `npm run ci` and `just ci` both passed there: 198/198 corpus parses, three highlight fixtures with 2/3/9 assertions, generator/Neovim checks, exact capture checks and 17 shared-capture assertions. A fresh `npm ci --offline` failed compiling tree-sitter's native dependency under Node 24 before CI. For both passing CI commands the clean worktree temporarily linked to the original checkout's existing project-local `node_modules`; the link was removed and clean status confirmed afterward. This verifies committed source with those dependencies, not a fresh install.
- Mode source: local branch `emacs-grammar-2026-10-20`, commit `5eb4f8fb6b5b9c737b47b4894351e781a78654a6` (message-only amend of `547a56d` to name the full `/home/danknutson/.agents/skills/testing-emacs-daemons/scripts/private-daemon` path). One commit contains the approved Q02 callback adapter, candidate pin, and S06 rule/test. Mode checkout is clean; branch is not published. The commit message records a repeatable private-daemon build/check/load/test/cleanup recipe.
- Private Emacs 30.1 verification: compile `src/parser.c` and `src/scanner.c` from the locally tagged clean worktree into a private `libtree-sitter-roc.so`, set `treesit-extra-load-path` to that private directory, and load the exact checked mode source. `symbol-file` resolved to the source checkout. The `f = |x| x?` tree contained `try_operator`, and `tests/all_syntax_test.roc` had zero `ERROR` nodes. The edited Elisp passed the checker; focused S06 plus neighboring imenu ERT passed 2/2, zero unexpected/skipped. The private daemon was stopped and cleaned. This local build was **not** a clone/install from the remote tag.
- S06's after-header branch now recognizes `module_exposes`; the separate generic rule indents the closing bracket. The strengthened normal-mode test verifies correct header, following declaration and import layout, explicit parents, and that the module-header declaration avoids generic `treesit-indent`. Verified in-memory mutation back to `exposes_list` makes that test fail on the fallback-path assertion (one unexpected, zero skipped), although generic fallback gives the same visible column zero in the tested valid fixture. Do not claim a demonstrated visible misindentation; see [S06 evidence](../side-findings/module-header-indentation.md#execution-evidence-local-unpublished).
- Intermediate blocker: a targeted font-lock ERT failed 1/1 on the stale `(where)` query with `treesit-query-error`. Mode initialization itself created a parser and font-lock settings. E02 must migrate font-lock, E03 structural Elisp, and full ERT must pass with zero unexpected results and zero skipped required font-lock tests before mode publication. Q03 is independent.
- **Published grammar and actual installer (E01 complete):** after authorization, `git push fork refs/heads/master:refs/heads/master refs/tags/2026-10-20.01:refs/tags/2026-10-20.01` succeeded without force, upstream setting, or mode push. `git ls-remote --symref fork HEAD refs/heads/master refs/tags/2026-10-20.01` confirmed default branch `master`, HEAD/master/tag each `e13e92b3ee585a00ac34c00372b9f8233e933cad`. A fresh plain Emacs 30.1 private daemon (PID `3828993`, socket `/tmp/emacs-test.ikNA4wqE/server`, isolated HOME `/tmp/emacs-test.ikNA4wqE/home`) initially reported Roc unavailable. It set `treesit-language-source-alist` to `(roc "https://github.com/Giesch/tree-sitter-roc/" "2026-10-20.01")`, `treesit-extra-load-path` to `("/tmp/emacs-test.ikNA4wqE/grammar")`, and called `(treesit-install-language-grammar 'roc "/tmp/emacs-test.ikNA4wqE/grammar")`. The installer returned successfully and wrote `/tmp/emacs-test.ikNA4wqE/grammar/libtree-sitter-roc.so` (945160 bytes); Roc then became available. The installer removes its temporary Git checkout, so no post-install checkout hash was available; provenance is the exact remote source/tag recipe, verified remote tag hash, initially unavailable language, successful installer call, private output library, and successful parse. The installed grammar's `f = |x| x?` tree contained `try_operator`; `tests/all_syntax_test.roc` (9896 bytes) had zero `ERROR` nodes. After loading the standalone checker, both mode/test Elisp files passed before exact-source load; `symbol-file` resolved the mode and indentation functions to the source checkout. Focused S06 and neighboring imenu ERT passed 2/2, completed 2, zero unexpected, zero skipped. An initial parser eval form failed on a missing closing parenthesis; the corrected form passed, so that failed call is not parse evidence. The private daemon PID `3828993` was stopped (socket refused) and test root `/tmp/emacs-test.ikNA4wqE` was cleaned.
- **Remaining mode gates:** source branch `emacs-grammar-2026-10-20` remains local; the published mode, Doom pin, installed grammar, and daily editor are unchanged. E02 must fix stale `(where)` font-lock rules, E03 must finish structural migration, and the full mode ERT suite must pass with zero unexpected results and zero skipped required font-lock tests before mode publication. S06's full-suite publication gate also remains open.

## Out of scope

- Font-lock changes (plan 02).
- Publishing the mode (plan 04).

## Verification

- `git rev-parse 2026-10-20.01`, fork HEAD/master, and fork `refs/tags/2026-10-20.01` all returned `e13e92b3ee585a00ac34c00372b9f8233e933cad` after the push.
- In separate disposable daemons with the locally built library and the actual remote-tag installed library, buffer parsing `f = |x| x?` returned a tree containing `try_operator`. `treesit-parse-string` is not available in the tested Emacs 30.1.
- E02/E03 and full mode ERT are later gates, not claims from the focused E01 check.

## Done when

- [x] Following authorization, grammar `master` and tag pushed to `fork`; remote HEAD/master/tag each confirmed at `e13e92b3ee585a00ac34c00372b9f8233e933cad`.
- [x] Local mode branch has the candidate pin and S06 in the same commit; not published.
- [x] The first local mode commit message contains the full private-daemon script path and repeatable local-build steps.
- [x] A new isolated daemon installed the **remote** published tag with `treesit-install-language-grammar`, verified private-library provenance, `try_operator`, zero fixture `ERROR` nodes, and focused ERT 2/2 without skips. Full mode ERT remains a later publication gate.
