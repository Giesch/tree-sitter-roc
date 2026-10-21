# E04: Publish the mode

Status: done (roc-ts-mode `7558c81f94e3dcd959d7ddce95892e8b5019a15c`, published fork `main` and mode tag `2026-10-21.01`; daily installation verified 2026-10-21).

## Execution evidence (2026-10-21)

- Fresh plain-Emacs disposable daemon installed remote grammar tag `2026-10-20.02` privately and loaded exact mode source. Full ERT: **31 total/completed, zero unexpected, zero skipped**. Test PID 1222112, socket `/tmp/emacs-test.4KkDg90N/server`; daemon stopped, socket refusal verified, test root removed.
- Operator approved fast-forwarding fork `main`, date/version tagging, and daily installation/activation. Remote `main` and new mode tag `2026-10-21.01` both verified at `7558c81f94e3dcd959d7ddce95892e8b5019a15c`. Grammar tag remains `2026-10-20.02`.
- Updated Doom `packages.el` Roc commit pin and `+roc.el` grammar-tag override. Both passed the Lisp checker in the private daemon. Unrelated staged/unstaged dotfiles edits were preserved; no dotfiles commit or broad sync was performed.
- Updated only Roc via daily Doom's straight.el fetch, public commit-checkout wrapper, and package rebuild. Installed checkout verified at the pinned commit, with no local changes. An initial direct Git-backend helper call failed; the public wrapper resolved it.
- Initial daily installation selected the old in-memory grammar source. Corrected that source and installed `.02`. The process retained the old native grammar and query activation failed; the operator restarted Doom and ran `server-start`.
- Fresh daily process PID 1226080 loaded installed `roc-ts-mode.elc`. In `*Roc E04 acceptance*`, `module [main]` plus `main = |x| x?` parsed without `ERROR`; `?` exposed `try_operator` and `roc-ts-question-mark-face`, with 47 font-lock settings. No pre-existing Roc buffers required refreshing. The persistent Doom override and live source entry now both select `.02`.
- No new graphical screenshot/navigation acceptance was claimed; daily face/node checks were programmatic.

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

- [x] Mode pushed and Doom pin updated.
- [x] Daily editor uses the new grammar and queries.
