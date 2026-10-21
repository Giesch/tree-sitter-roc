# Side findings

## Current state — 2026-10-21

S01–S03 are complete. S06 is complete and shipped in mode `7558c81` through E04; its full-suite and daily-installation gates are closed. S05 is complete: plain-import highlight precedence was reproduced and fixed, with seven new assertions and passing CI. S04 remains planned and should extend S05's coverage to additional syntax forms.

The original prerequisite changes below were completed before the numbered grammar plans.

| Plan | Repository | Change |
|---|---|---|
| [module-header.md](module-header.md) | tree-sitter-roc | `module [f]` parses as `ERROR` |
| [imenu-nominal-types.md](imenu-nominal-types.md) | roc-ts-mode | imenu does not list nominal types |
| [remove-grammar-compat.md](remove-grammar-compat.md) | roc-ts-mode | Remove grammar-version probes and test skips |

`imenu-nominal-types.md` and `remove-grammar-compat.md` both edit `tests/roc-ts-mode-test.el`. Do them one after the other, not in parallel.

The grammar plans start from the commit that contains `module-header.md`.

## Follow-up plans

These are separate from the three prerequisite side findings above. They do not block the existing numbered grammar plans.

| Plan | Repository | Change |
|---|---|---|
| [module-exposure-forms.md](module-exposure-forms.md) | tree-sitter-roc | Verify and support wildcard, renamed, and qualified items in module exports and import exposure lists |
| [import-exposure-highlights.md](import-exposure-highlights.md) | tree-sitter-roc | Test and, if needed, fix plain import-exposure capture precedence |
| [module-header-indentation.md](module-header-indentation.md) | roc-ts-mode | Update module-header indentation together with the later grammar pin |

Prefer the plain-import highlight follow-up before the richer exposure forms; reuse its tests rather than duplicating them. Run grammar changes separately because they regenerate the same parser files. The completed indentation follow-up shipped with E01's pin update and the final E04 release; do not reimplement it.
