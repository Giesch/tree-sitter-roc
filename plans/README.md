# Plans

These plans change tree-sitter-roc and roc-ts-mode. The goal is shared, directly queryable syntax and generated editor queries. On Emacs 30.1 with libtree-sitter 0.26, one mode-local derived-name refinement remains necessary because query predicates are incompatible with that runtime.

Each leaf plan is self-contained. Read [context.md](context.md), the README of the plan's directory, and the leaf plan. Do not read other plans.

## Current state

| Work | State | Next action |
|---|---|---|
| Grammar 01–07 and queries 01–02 | Complete; published, including E02's face-only query update in `2026-10-20.02` | No remaining prerequisite work |
| E01 grammar pin and E02 generated-query integration | Complete locally; mode commit `f258f2f` is not pushed | Await E04 publication |
| E03 structural Elisp | Complete and locally committed at mode `7558c81`; recorded full ERT 31/31 | Await E04 publication |
| E04 mode publication and daily installation | Pending; separately approval-gated | Obtain approval and follow publication/daily-installation checks |
| Query 03 and S04/S05 | Independent pending follow-ups | Query 03 can proceed independently; S05 precedes S04 |

Grammar plans **01–07 are complete and reviewed**. Query plans **01–02 were committed** at tree-sitter-roc `e13e92b3ee585a00ac34c00372b9f8233e933cad`; E02's face-only query-contract update is `6d85644f993edc543be0214bfffe60905d8e66d1`, separately approved and published on fork `master` and tag `2026-10-20.02` (remote refs verified). E01's earlier `2026-10-20.01` tag remains unchanged. The mode source branch `emacs-grammar-2026-10-20` contains the E01/S06 local commit `5eb4f8fb6b5b9c737b47b4894351e781a78654a6` and E02 local commit `f258f2f674541271f998641686441d0c1b0cf803`; it was not pushed, and the published mode and daily editor still use grammar `2026-10-13.01`. E01–E03's local implementation and verification are complete. E03 is locally committed at `7558c81f94e3dcd959d7ddce95892e8b5019a15c` after `f258f2f`; E04 mode publication and daily installation remain pending.

- **E03 complete locally (2026-10-21):** [Structural Elisp](emacs-mode/03-structural-elisp.md#execution-evidence-2026-10-21-local-unpublished) is implemented and verified, locally committed at mode `7558c81f94e3dcd959d7ddce95892e8b5019a15c`: full ERT 31/31, zero unexpected/skipped, strict compilation and Lisp checks passed. Field-based name lookup, mutable-declaration selection/imenu, and sibling-annotation indentation are covered by regression tests. The syntax fixture's programmatic imenu index contains all five expected categories, with no parse errors. The private daemon and installation were cleaned up. **Next:** [E04: mode publication](emacs-mode/04-publish.md) still requires its own approval and checks before pushing, changing Doom, or touching the daily editor. [E02](emacs-mode/02-consume-generated-queries.md) is locally committed at `f258f2f674541271f998641686441d0c1b0cf803`: the remotely published `.02` query tag installed privately, strict four-file compilation passed with real sibling autoloads and no warning suppression, ERT passed 29/29 (zero unexpected/skipped), 47 ordered settings and deterministic generation passed. Paired old installed-mode and candidate screenshots were inspected at matching 110×43 geometry; visual evidence covers highlighting/layout, not the navigation subcheck, whose evaluation failed to execute. [Query plan 03: highlight tests](queries/03-highlight-tests.md) proceeds independently.
- **E01 complete (2026-10-20):** fork `master` and tag `2026-10-20.01` were verified remotely at `e13e92b`. A fresh disposable Emacs 30.1 daemon installed the published tag with `treesit-install-language-grammar` into a private directory. Roc was unavailable before installation and available afterward; the installed grammar exposed `try_operator` and parsed `tests/all_syntax_test.roc` with zero `ERROR` nodes. Exact-source loading and Lisp checks passed; S06 indentation and neighboring imenu ERT passed 2/2 with zero unexpected results and zero skips. The daemon and private installation were cleaned up; the daily editor, Doom pin, and installed daily grammar were untouched.
- **S06 implementation and review:** the pin, module-header rule migration, and focused tests share local mode commit `5eb4f8f`. The changed condition controls indentation after the header; an unchanged generic rule aligns its closing bracket. The test detects restoration of the old node condition through normal-command rule selection, not a visible indentation difference: both paths produced column zero. Spec, plan-shape, Tokenparty Fable, and Kimi-at-`high` reviews completed with no unresolved Critical/Important findings after corrections. The subsequent E03 full suite passed 31/31; publication and daily-editor acceptance remain gated by E04.
- **E02 verification:** the old handwritten `(where)` font-lock query was removed. The new mode uses 46 ordered generated sections plus one final derived-name override (47 settings); the full private mode ERT passed 29/29 with zero unexpected results and zero skips on grammar tag `.02`. E03's subsequent structural migration has its own 31/31 full-suite result; E04 publication remains pending.
- **Q01 delivered:** shared highlight parity, editor-specific capture mappings, regenerated Helix/Neovim/Zed queries, and a shared-capture regression guard included in both CI entry points.
- **Q02 delivered (historical contract):** object-valued capture maps, ordered feature sections, an Emacs highlight map, and 46 generated sections covering the mode's 24 features. Its former generated callback capture was superseded by E02's published face-only `.02` query mapping and one separate mode-local override. That integration is locally committed, not published in the mode.
- **Current query verification (`6d85644f`, tag `2026-10-20.02`):** normal `npm run queries:check` and `npm run ci` passed after commit, without the temporary-index workaround used during review. CI verified 198 corpus parses, all three highlight fixtures, 17 shared-capture assertions, Neovim checks, and exact capture regressions. A fresh private Emacs 30.1 daemon passed **10/10 runtime ERT tests**, zero unexpected results and zero skips, independently of loading roc-ts-mode. The suite verifies all generated captures resolve to faces, exact inferred-name spans, feature membership and toggles, and both Roc-specific faces. Fable and Kimi approved the final query and mode implementations with no unresolved Critical/Important findings. The earlier nine-test Q02 result is historical, not the current suite count.
- **Release CI (2026-10-20):** `npm run ci` and `just ci` both passed at committed `e13e92b` in a separate clean release worktree. Each run verified 198/198 corpus parses, all three highlight fixtures (2, 3, and 9 assertions), generated queries, and the query/capture guards (including 17 shared-capture assertions). The worktree was clean after both runs. `npm ci --offline` failed building the tree-sitter native dependency under Node 24 before CI; both passing commands used a temporary symlink to the original checkout's existing project-local `node_modules`, removed after each run. Thus CI passed against the committed source, but a fresh dependency installation under this Node version was not demonstrated.
- **Approved Q02 adjustments (historical; capture contract superseded by E02):** existing-editor generated byte identity was relaxed in favor of preserving editor semantics while splitting captures and correcting pattern order. Emacs 30.1/libtree-sitter 0.26.0 accepts predicate syntax at compilation but rejects it during evaluation. E02's `.02` query artifact omits predicate-bearing refinements and contains only face captures; the locally committed mode appends one callback rule.
- **Completed grammar follow-ups:** feature-preserving parser-size reduction, pinned generator and metadata checks, exact query-capture regression checks, and highlight-fixture completion/failure checks.
- **Final parser metrics and ceilings:** 3,133 states, 903 large states, and 5,213,834 bytes. The size-reduction follow-up saved 7,609 bytes, two states, and two large states relative to the post-plan-07 baseline, with tested parse trees and captures preserved.
- **Verification boundary:** E02 query faces, feature toggles, full local mode ERT, and graphical highlighting/layout were tested in isolated Emacs; six paired PNGs and readable snapshots are recorded in [E02](emacs-mode/02-consume-generated-queries.md). The graphical navigation subcheck did not execute and is not counted as passed. E03 separately verified declaration-name lookup, defun selection, imenu indexing, and indentation in plain Emacs, with full ERT 31/31. Interactive graphical navigation and E04 daily-editor acceptance remain unverified. A historical optional mutation probe timed out and was not counted as verification.

## Repositories

| Path | Contents |
|---|---|
| `~/Projects/tree-sitter-roc` | Grammar (`grammar.js`), external scanner (`src/scanner.c`), corpus tests (`test/corpus/`), highlight tests (`test/highlight/`) |
| `~/Projects/tree-sitter-roc/queries/` | Base queries. Capture names follow Helix. These files are the source of truth for highlighting. |
| `~/Projects/tree-sitter-roc/query-maps/` | One JSON map per editor. It maps base capture names to editor capture names. |
| `~/Projects/tree-sitter-roc/queries-generated/` | Generated per-editor queries. `tools/template-queries.js` writes them. Do not edit by hand. |
| `~/Projects/roc-ts-mode` | Emacs major mode. Remote and pin rules are in the `roc-ts-mode-maintenance` skill. |

## Shared commands (grammar repository)

Use the repository-local Tree-sitter CLI pinned in `package.json` (currently 0.25.10), not an ambient global CLI. The commands below use the pinned version.

- `just test-gen`: regenerate the parser and run the CLI tests; use a CI command below for guarded completion verification.
- `just test-update`: rewrite corpus expectations. Review the full diff; intentional-error fixtures may require manual expectation updates.
- `just parser-metrics-check`: fail when parser states or size exceed the committed budget in `tools/check-parser-metrics.js`.
- `just queries-generate`: regenerate `queries-generated/`.
- `npm run ts:test`: run corpus inventory checks and the test wrapper that verifies every highlight fixture completed.
- `just ci`: check generated-artifact format/version and cleanliness, parser metrics, corpus/highlight execution, generated queries, and expression-body capture regressions. Every grammar or query plan ends with a passing `just ci`.
- `npm run ci`: run the corresponding npm verification pipeline, including the corpus inventory guard.

For Emacs work, use the `emacsclient` and `testing-emacs-daemons` skills. Never use the `emacs` binary directly.

## Order

1. **Done:** [side-findings/](side-findings/README.md): the original three prerequisite plans (S01–S03).
2. **Done:** [grammar/](grammar/README.md): plans 01 to 07 completed in order, with recorded follow-ups. Each plan regenerated `src/parser.c`.
3. **Committed and published with the grammar:** [queries/](queries/README.md): plans 01 and 02 at `e13e92b`, with approved scope adjustments and passing clean-worktree CI recorded above.
4. **E01–E03 locally complete; mode unpublished:** [emacs-mode/](emacs-mode/README.md): grammar query tag `2026-10-20.02` is published at `6d85644f993edc543be0214bfffe60905d8e66d1`. E02 mode integration is locally committed at `f258f2f674541271f998641686441d0c1b0cf803`; E03 is locally committed after it at `7558c81f94e3dcd959d7ddce95892e8b5019a15c`. E03's private remote-tag installation, strict compilation, Lisp checks, full ERT 31/31 (zero unexpected/skipped), and programmatic fixture imenu check passed. E02's paired graphical highlighting/layout inspection passed; graphical navigation was not verified. E03's commit is complete; E04 must re-run isolated tests before publication. E04 publication/daily installation requires separate approval and remains pending.
5. **Pending:** [queries/03-highlight-tests.md](queries/03-highlight-tests.md): at any point after query plan 01. No Emacs-mode plan needs it.
6. **Planned follow-ups:** [side-findings/](side-findings/README.md#follow-up-plans): the tasks added after the original three side findings. Do [S05: import-exposure highlights](side-findings/import-exposure-highlights.md) before [S04: additional exposure forms](side-findings/module-exposure-forms.md), reusing S05's tests. [S06: after-header indentation](side-findings/module-header-indentation.md) is locally committed with E01's pin in item 4, not published. These follow-ups do not block the numbered grammar plans.

## Compatibility policy

- roc-ts-mode supports exactly one grammar: the tag it pins in `treesit-language-source-alist`. The unpublished mode source branch pins published grammar/query tag `2026-10-20.02`; the published mode and daily installation still use `2026-10-13.01`. Do not use the new mode source against the old installed grammar.
- The mode does not support older or newer grammars. Do not add runtime probes for grammar features, fallback rules, version checks, or test skips that depend on grammar features.
- A mode commit that changes the pin also changes every Elisp rule and test that depends on the grammar. The user reinstalls the grammar after updating the mode.
- Grammar changes reach the mode through [emacs-mode/01-grammar-tag-and-pin.md](emacs-mode/01-grammar-tag-and-pin.md). Query plans update base/generated queries, maps, generator code, and regression tests; E02 embeds the face-only Emacs output in the mode. Q02's generated callback capture is historical; E02's published `.02` face-only query and locally committed separate mode-local refinement rule replace it. E03 structural migration is complete and locally committed at `7558c81`; E04 mode publication remains pending.
- tree-sitter-roc does not keep node names for compatibility. A grammar plan renames or removes nodes when the plan requires it.

## Leaf plan template

Every leaf plan has these sections: Goal, Depends on, Context, Steps, Out of scope, Verification, Done when.

## Status

The original three side findings are complete:

| Plan | Completion commit |
|---|---|
| [S01: Module header](side-findings/module-header.md) | tree-sitter-roc: `b1375d35e851e0b40c1daf1b15e02c0d06d0758c` |
| [S02: Imenu nominal types](side-findings/imenu-nominal-types.md) | roc-ts-mode: `4ca916aaf5ada32dd1c77f7ad42028a5015c19c0` |
| [S03: Remove grammar compatibility](side-findings/remove-grammar-compat.md) | roc-ts-mode: `f17d62f8aa9f92fd0d767226d34d176d89ffa3ec` |

Grammar plans 01–07 are done (completion commits `e4b700c`, `85f333e`, `7d14921`, `35b9fc3`, `e4f8365`, `10d42dc`, `c0f87e4`; see [grammar/](grammar/README.md) for follow-ups). Query plans 01–02 were committed at tree-sitter-roc `e13e92b`; their historical checklists retain approved scope adjustments. E01's fork tag `2026-10-20.01` remains there; E02's face-only query update is published at `6d85644f993edc543be0214bfffe60905d8e66d1` on fork `master` and tag `2026-10-20.02`. Local roc-ts-mode E01/S06 commit `5eb4f8fb6b5b9c737b47b4894351e781a78654a6` is followed by E02 commit `f258f2f674541271f998641686441d0c1b0cf803`; the mode has not been pushed. E02's full private ERT and graphical highlighting/layout comparison passed, but the graphical navigation subcheck did not execute and is not a passed test. E03 is independently implemented and verified, and locally committed at `7558c81f94e3dcd959d7ddce95892e8b5019a15c` after `f258f2f`: full ERT 31/31, zero unexpected/skipped, strict compilation, Lisp checks, and programmatic fixture imenu checks passed. Query plan 03, E04 publication, and daily installation remain pending; S04/S05 remain planned.

Mark a plan as done by adding `Status: done (<commit>)` under its title, and update the summary above and the order list.
