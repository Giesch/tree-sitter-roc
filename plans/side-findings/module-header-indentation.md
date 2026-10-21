# S06: Migrate module-header indentation with the grammar pin

Status: locally implemented and tested in unpublished roc-ts-mode `5eb4f8fb6b5b9c737b47b4894351e781a78654a6`; full-suite and publication gates remain open.

## Goal

When roc-ts-mode adopts a grammar containing S01, its after-header indentation condition recognizes a closing `]` whose parent is `module_exposes` and indents the following line at column zero. The independent generic closing-bracket rule continues to indent `]` itself. The mode supports exactly its pinned grammar.

## Depends on

- S01, `module-header.md`, which changed the module-header list node from `exposes_list` to `module_exposes`.
- Execute as part of [../emacs-mode/01-grammar-tag-and-pin.md](../emacs-mode/01-grammar-tag-and-pin.md), against that task's candidate grammar tag. Include the indentation change and tests in the same mode commit as the pin update, not as an earlier change against the old pin.
- The existing mode migration also changes font-lock rules in E02. If stale font-lock queries prevent mode initialization under the candidate grammar, prepare the necessary E02 changes before running mode-level tests. Do not bypass initialization errors or add compatibility fallbacks merely to run this test.

## Context

- The mode's after-header indentation condition previously checked whether the preceding closing bracket's parent was `exposes_list`; S06 changes that check to `module_exposes`. The generic closing-bracket indentation rule is separate and unchanged.
- S01's grammar uses `module_exposes` for module headers. Imports retain their separate exposure syntax. Changing every occurrence of `exposes_list` would be incorrect.
- The changed condition runs while indenting the line after a header: it inspects the closing bracket on the previous nonblank line. Generic closing-bracket indentation handles the bracket itself. Under the pinned grammar the previous `]` belongs to `module_exposes`, so the old `exposes_list` condition does not match. In a valid multiline header followed by a declaration, the old rule falls through to `treesit-indent`, which also gives column zero. Test the selected normal-mode path directly; do not claim this fixture has a different visible indentation under the old rule.
- Source checkout: `/home/danknutson/Projects/roc-ts-mode`. Load `roc-ts-mode-maintenance`, `emacsclient`, and `testing-emacs-daemons` for implementation and verification. Preserve the daily editor and its installed grammar.
- Compatibility policy in `plans/README.md`: grammar-dependent mode changes accompany the pin update. Do not support both node shapes through probes, fallback rules, or grammar-dependent test skips.

## Steps

1. Inspect the current indentation rule and existing ERT/indentation fixture harness. Confirm the candidate grammar tag includes S01 and inspect a multiline module header's parse tree to establish the closing bracket's actual parent.
2. Cover a valid multiline module header with lowercase and uppercase exports, a trailing comma, and a following declaration. Indent an incorrectly placed closing bracket and, separately, an incorrectly indented declaration after a correct bracket. Check expected columns after the normal mode indentation command. The closing bracket is a layout control; the declaration exercises the changed after-header condition.
3. Add a stability check for an already correctly indented header and a neighboring import exposure-list control. Confirm the bracket's actual parent. If richer forms from S04 are included in the candidate tag, include a representative supported form without making S04 a prerequisite.
4. Update only the module-header indentation condition to match `module_exposes`. Keep unrelated import/list rules intact. Assert that the normal mode command selects the after-header rule instead of generic `treesit-indent` for the following declaration. Mutate the rule to the old condition for a separate test and record branch-selection and visible-output results independently.
5. Check edited Elisp before loading. Load the exact source in a disposable daemon, verify `symbol-file`, and use a private grammar installation built from the candidate tag. Confirm the loaded grammar's provenance; do not rely on the daily editor's installed binary.
6. Run focused indentation tests. Coordinate with E02/E03 for any broader migration prerequisites, then run the full ERT suite with zero unexpected results and zero skipped font-lock tests before publishing the mode. Record any intermediate test blocker explicitly rather than marking verification complete.
7. Include this change and its tests in the mode pin-update commit. Record the candidate tag, test results, and any remaining migration work in the E01 execution notes. Publishing remains separately approval-gated.

## Execution evidence (local, unpublished)

- Candidate local tag `2026-10-20.01` points to grammar commit `e13e92b3ee585a00ac34c00372b9f8233e933cad`. `npm run ci` and `just ci` passed in a clean release worktree using the original checkout's existing project-local dependencies through a temporary symlink. Fresh `npm ci --offline` failed compiling a native dependency under Node 24; no fresh dependency-install claim. A private Emacs 30.1 daemon loaded a library built from the local tagged commit and exact mode source, not a remote-tag installation. The valid header bracket is `]` under `module_exposes`; import exposure remains `exposes_list`.
- Focused ERT with the new rule passed (one test, zero unexpected, zero skipped). It checks misplaced and stable closing brackets, a misindented following declaration, an already correct following declaration, the import control, and explicit bracket parents. The following declaration reaches the special after-header branch without calling `treesit-indent`.
- A verified in-memory substitution of `module_exposes` with `exposes_list` in `roc-ts--indent-line` makes the focused test fail at `(should-not fallback-called)` (one unexpected, zero skipped). Without that branch assertion, the behavior fixture passed under the old rule: `treesit-indent` also gave the declaration column zero. This is a rule-selection regression, not demonstrated visible misindentation.
- Focused S06 and neighboring imenu ERT passed 2/2 (zero unexpected/skipped) using a local tagged-source build; edited Elisp passed the checker before exact-source loading. Mode source pin, S06 rule/test, and approved Q02 callback adapter share local mode commit `5eb4f8fb6b5b9c737b47b4894351e781a78654a6` (message-only amend of reviewed `547a56d` for the full private-daemon script path). A targeted font-lock ERT failed 1/1 on stale `(where)` with `treesit-query-error`; E02, E03, and full mode ERT with zero unexpected and zero skipped required font-lock tests remain before mode publication.
- After the authorized grammar push, fork default `master` and tag `2026-10-20.01` both resolved to `e13e92b3ee585a00ac34c00372b9f8233e933cad`. In a **fresh** private Emacs 30.1 daemon, `(treesit-install-language-grammar 'roc "/tmp/emacs-test.ikNA4wqE/grammar")` with source `https://github.com/Giesch/tree-sitter-roc/` at tag `2026-10-20.01` installed an isolated library and made Roc available. Installed-grammar `f = |x| x?` contained `try_operator`; `all_syntax_test.roc` had zero `ERROR` nodes. Both Elisp files passed the checker; exact-source S06 plus imenu ERT passed 2/2, zero unexpected/skipped. The mode branch remains unpublished; no daily editor, Doom, or installed daily grammar update occurred. The E01 notes contain the complete installer procedure/provenance and cleanup status.

## Out of scope

- Updating the daily editor, Doom package pin, or installed dependency checkout.
- A runtime grammar-version detector, dual-node fallback, or conditional test skip.
- General indentation redesign, import syntax changes, or unrelated font-lock changes.
- Independent permission to publish the mode or update the daily editor. E01's grammar push is complete; E04 mode publication remains separately gated.

## Verification

- Parse inspection confirms the tested closing bracket belongs to `module_exposes` under the candidate grammar.
- The incorrectly indented closing bracket moves to the expected column; re-indenting the correct fixture makes no change.
- The import exposure-list control retains its expected indentation.
- Edited Elisp passes checks before exact-source loading in a disposable daemon.
- Focused indentation tests pass against the candidate pin. Full ERT passes after the required mode migration changes, with zero unexpected results and zero skipped font-lock tests.
- The mode pin update and indentation rule/test changes are committed together. No old-node fallback or grammar probe is introduced.

## Done when

- [x] The after-header indentation condition uses `module_exposes` while the generic bracket rule remains separate.
- [x] Focused layout, rule-selection, stability, and import-control tests pass with both local tagged-source and actual remote-tag installed grammar. The old condition fails the rule-selection assertion but produces the same visible column in the tested fixture.
- [ ] Full-suite migration verification passes after E02/E03, before mode publication.
- [x] The unpublished pin-update commit contains the rule change and tests; remote grammar push and E01 installer check are complete. Mode publication remains open.
