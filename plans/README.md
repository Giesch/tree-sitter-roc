# Plans

These plans change tree-sitter-roc and roc-ts-mode to provide directly queryable syntax and shared generated editor queries.

Each leaf plan is self-contained. Read [context.md](context.md), its directory README, and the leaf plan. Historical execution notes describe the state at the time of each test; the current status below supersedes intermediate publication gates.

## Current state — 2026-10-21

| Work | State | Next action |
|---|---|---|
| Grammar 01–07 | Complete and reviewed | None |
| Queries 01–02 | Complete and published, including face-only Emacs queries in grammar tag `2026-10-20.02` | None |
| Emacs E01–E04 | Complete, published, installed, and verified in the daily editor | None |
| Side findings S01–S03 and S06 | Complete; S06 shipped with the mode | None |
| Query 03 | Complete; five topic fixtures and failure detection verified | None |
| S05 import-exposure highlighting | Complete; precedence failure reproduced and fixed | None |
| S04 additional exposure forms | Complete; versioned syntax, grammar, corpus, and highlighting verified | Release/pin update is separate |
| Where aliases W01–W03 | Not started; W01 and W02 are required for Builtin.roc to parse clean | Do W01 |

### Published and installed versions

- Grammar/query release: `2026-10-20.02` at tree-sitter-roc `6d85644f993edc543be0214bfffe60905d8e66d1`. Earlier tag `2026-10-20.01` remains unchanged at `e13e92b3ee585a00ac34c00372b9f8233e933cad`.
- Mode: fork `main` and mode tag `2026-10-21.01` both verified at `7558c81f94e3dcd959d7ddce95892e8b5019a15c`. This includes E01/S06 `5eb4f8f`, E02 `f258f2f`, and E03 `7558c81`.
- Doom's mode commit pin and installed checkout use `7558c81`; its grammar override and live source entry use `2026-10-20.02`. Only the Roc package was rebuilt. Dotfiles edits were left uncommitted and unrelated changes preserved.
- Grammar releases use immutable `YYYY-MM-DD.NN` tags, not commit SHAs in `treesit-language-source-alist`. Doom's mode package pin remains a full mode commit hash. The maintenance skill now documents this distinction; grammar releases do not require mode tags.

### Verification and boundaries

- E04 fresh isolated full ERT: **31 total/completed, zero unexpected, zero skipped**, loading exact mode source and a private installation of the published `.02` grammar tag. Private daemon and installation cleaned up.
- Daily activation initially failed because the running process retained the old native grammar. After an operator-performed restart, installed mode acceptance passed: `?` has `roc-ts-question-mark-face`, its node is `try_operator`, the sample has no `ERROR` nodes, and 47 font-lock settings are loaded. No pre-existing Roc buffers required refresh.
- Both edited Doom Lisp files passed the Lisp checker. No broad package sync, dotfiles commit, or editor restart by the agent occurred.
- Earlier E02 paired graphical highlighting/layout inspection passed. Its graphical navigation subcheck did not execute. E03 programmatic declaration lookup, defun selection, imenu, and indentation passed. E04 daily acceptance was programmatic, not a new screenshot/navigation check.
- Grammar/query CI at `.02` passed: 198 corpus parses, three highlight fixtures, 17 shared-capture assertions, generated-query and exact-capture checks. Independent private Emacs query-runtime ERT passed 10/10, zero unexpected/skipped.
- Release CI at `e13e92b` passed both npm and just entry points in a clean worktree using existing project-local dependencies. Fresh `npm ci --offline` under Node 24 failed building a native dependency; fresh dependency installation was not demonstrated.
- S04's reviewed, operator-approved parser ceilings are 3,144 states, 903 large states, and 5,227,711 bytes. Final size is 5,227,026 bytes. Do not refresh budgets automatically.

Detailed evidence: [E04 publication and installation](emacs-mode/04-publish.md), [E02 query integration](emacs-mode/02-consume-generated-queries.md), [E03 structural migration](emacs-mode/03-structural-elisp.md), and individual grammar plans.

## Remaining work and order

[Q03: Highlight tests](queries/03-highlight-tests.md) is complete. `just ci` passed with 198 corpus parses and all seven highlight fixtures; a deliberately wrong capture failed as expected and was restored.

[S05: Import-exposure highlights](side-findings/import-exposure-highlights.md) is complete. The original parent capture lost to `variable`; capturing identifier children fixes the regression. `just ci` passed with 198 corpus parses and all eight highlight fixtures.

[S04: Additional exposure forms](side-findings/module-exposure-forms.md) is complete against Roc revision `37daf70979eae0cbf6bc603ab9b6721076ed0654`: qualified names, wildcards, and item aliases have corpus/highlight coverage. Pinned-CLI `just ci` and `npm run ci` passed with 220 corpus parses and all ten highlight fixtures. Publishing and downstream grammar/query pin updates remain separate work.

[Where-alias plans](where-aliases/README.md): W01 and W02 make Roc `src/build/roc/Builtin.roc` parse without `ERROR` nodes. The Zed outline test (faldor20/tree-sitter-roc#39) uses that file as its fixture. Do W01 and W02 before the outline work. W03 is not needed for Builtin.roc.

Grammar work regenerates shared parser files; execute separate grammar changes serially. These follow-ups do not reopen the completed E01–E04 release.

## Repositories and commands

- `~/Projects/tree-sitter-roc`: grammar, scanner, corpus/highlight tests, base queries, capture maps, and generated editor queries.
- `~/Projects/roc-ts-mode`: editable mode source; never maintain code in Doom's installed dependency checkout.
- Base `queries/` files are authoritative; regenerate `queries-generated/` rather than editing generated output.
- Use the project-local Tree-sitter CLI pinned in `package.json` (0.25.10), not an ambient global CLI.
- `npm run ci` / `just ci`: generated-artifact checks, parser metrics, corpus/highlight completion guards, generated queries, and capture regressions.
- `npm run ts:test`: guarded corpus and highlight tests. Bare CLI success can miss later highlight assertions.
- `just test-gen`: regenerate and test; `just test-update`: rewrite corpus expectations, requiring diff review.
- `just queries-generate`: regenerate editor queries; `just parser-metrics-check`: enforce parser budgets.
- For mode work, load `roc-ts-mode-maintenance`, `emacsclient`, and `testing-emacs-daemons`. Use private daemons for tests; daily activation requires operator coordination.

## Compatibility and completion

The mode supports exactly its pinned grammar. Do not add grammar-feature probes, compatibility fallbacks, or feature-dependent test skips. A mode release changing the grammar pin must include all dependent query and structural Elisp changes; the installed grammar must be updated too.

Emacs uses 46 generated face-only sections plus one local derived-name refinement (47 settings). The local callback remains necessary on Emacs 30.1/libtree-sitter 0.26 because predicate-bearing refinements cannot be evaluated by that runtime.

Completed groups: [grammar 01–07](grammar/README.md), [queries 01–02](queries/README.md), [Emacs E01–E04](emacs-mode/README.md), and [side findings S01–S03/S06](side-findings/README.md).

Every leaf plan has Goal, Depends on, Context, Steps, Out of scope, Verification, and Done when sections. When a plan completes, record `Status: done (<commit>)`, reconcile its checklist, and update the relevant summaries.
