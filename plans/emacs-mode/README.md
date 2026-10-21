# Emacs-mode plans

## Current state — complete (2026-10-21)

E01–E04 and S06 are delivered in mode `7558c81f94e3dcd959d7ddce95892e8b5019a15c`, published on fork `main` and mode tag `2026-10-21.01`. Doom's mode pin and installed checkout use that commit; both the mode and Doom's explicit grammar override select published grammar `2026-10-20.02`.

| Plan | State |
|---|---|
| [E01: Grammar tag and pin](01-grammar-tag-and-pin.md) | Done; initial pin/S06 commit `5eb4f8f`, final grammar tag `.02` adopted by E02 |
| [E02: Generated queries](02-consume-generated-queries.md) | Done at `f258f2f`, published in the final mode release |
| [E03: Structural Elisp](03-structural-elisp.md) | Done at `7558c81`, published in the final mode release |
| [E04: Publication and daily installation](04-publish.md) | Done; installed mode and daily grammar verified |

Fresh E04 isolated ERT passed **31/31**, zero unexpected/skipped. Only Roc was rebuilt through Doom's straight.el session; unrelated dotfiles edits were preserved and no dotfiles commit was made. After an operator-performed restart to clear the old native grammar, daily acceptance verified `try_operator`, `roc-ts-question-mark-face`, no sample parse errors, and 47 settings. No pre-existing Roc buffers needed refreshing. See [E04 evidence](04-publish.md#execution-evidence-2026-10-21).

E02's prior paired graphical highlighting/layout inspection passed; its graphical navigation subcheck did not execute. E03 separately verified declaration-name lookup, defun selection, imenu, and indentation programmatically. No new E04 graphical navigation acceptance is claimed. Intermediate local-only statements in leaf execution notes describe historical checkpoints, not outstanding release gates.

## Goal and architecture

roc-ts-mode uses pinned grammar and generated face-only queries from tree-sitter-roc. It builds 46 ordered generated sections plus one local derived-name refinement. The local callback is needed because Emacs 30.1/libtree-sitter 0.26 cannot evaluate the required predicates. Indentation, imenu, defun navigation, hideshow, and commands remain in Elisp.

## Maintenance contract

All package source edits belong in `~/Projects/roc-ts-mode`, not Doom's installed checkout. Load `roc-ts-mode-maintenance` before changes; use `emacsclient` and `testing-emacs-daemons` for Emacs operations.

The mode supports exactly its pinned grammar. Do not add runtime grammar probes, compatibility fallback rules, or grammar-dependent test skips. Grammar-dependent Elisp and queries must ship together with the pin change. Test against the published grammar tag before publishing the mode; daily installation and activation require operator coordination.

Grammar source entries use immutable `YYYY-MM-DD.NN` tags, not SHAs; update the Doom grammar override as well as the mode's entry. Doom's mode package pin remains a full mode commit hash. A grammar release does not require a mode tag.

## Remaining work

No Emacs-mode plan remains open. [Q03](../queries/03-highlight-tests.md) is also complete: topic-based highlight fixtures and assertion failure detection pass. Remaining independent grammar/query follow-ups are [S05](../side-findings/import-exposure-highlights.md), and then preferably [S04](../side-findings/module-exposure-forms.md).
