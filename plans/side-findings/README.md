# Side findings

Changes found during the grammar review. Do these plans first, before the grammar plans. They do not depend on any other plan.

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

Prefer the plain-import highlight follow-up before the richer exposure forms; reuse its tests rather than duplicating them. Run grammar changes separately because they regenerate the same parser files. The indentation follow-up is part of E01's mode pin update, not a change to make against the current old pin.
