# E04: Publish the mode

Status: pending; E01–E03 are locally committed through roc-ts-mode `7558c81f94e3dcd959d7ddce95892e8b5019a15c` on `emacs-grammar-2026-10-20`, pinning grammar `2026-10-20.02`. No mode push or daily-editor update has been performed. Re-run isolated ERT before requesting publication approval.

## Goal

The daily editor runs the mode branch from plans 01 to 03 with the new grammar.

## Depends on

[02-consume-generated-queries.md](02-consume-generated-queries.md) and [03-structural-elisp.md](03-structural-elisp.md).

## Context

The `roc-ts-mode-maintenance` skill defines every step: verify the fork branch with `git ls-remote --symref`, push, set the Doom `:pin` in `~/.dotfiles/.config/doom/packages.el`, update the installed package, and refresh open buffers. Follow it exactly. Publishing is outward-facing; ask the user before pushing.

## Steps

1. Run all ERT tests once more on the branch in a disposable daemon. Zero unexpected results.
2. Ask the user to approve the push. State the branch, the commits, and the grammar tag.
3. Follow the skill: merge or push the branch, update the Doom pin, update the installed package.
4. Install the new grammar in the daily editor: `M-x roc-ts-install-treesit-grammar`. Ask the user before you run it through `emacsclient` on the daily server.
5. Refresh open Roc buffers as the skill describes. `defvar` values do not reload; re-evaluate `roc-ts--generated-font-lock-queries` and rerun `roc-ts-mode` in each Roc buffer.

## Out of scope

- Further grammar changes.

## Verification

- The installed checkout is at the pinned commit.
- In the daily editor, a Roc buffer shows `?` in `roc-ts-question-mark-face`. Check with `(get-text-property (point) 'face)` on a `?`.

## Done when

- [ ] Mode pushed and Doom pin updated.
- [ ] Daily editor uses the new grammar and queries.
