# Q03: Highlight tests for base captures

## Goal

`test/highlight/` asserts the captures that editors depend on most. A grammar change that breaks one of these captures fails `tree-sitter test`.

## Depends on

[01-shared-query-parity.md](01-shared-query-parity.md).

## Context

- Existing example: `test/highlight/derived_methods.roc`. Assertions are comments with `^` under the source span and the capture name.
- `tree-sitter test` runs highlight tests with `queries/highlights.scm`.
- Highlight tests assert the highest-priority capture at each position.

## Steps

1. Add one file per topic. Each file asserts the captures listed:
   - `try_operator.roc`: postfix `?` and infix `?` as `keyword.control.return.roc-special.try`; `??` as `operator`; `return` as `keyword.control.return`.
   - `crash.roc`: `crash` as `keyword.control.exception`.
   - `tag_types.roc`: tag name in a tag union type as `type.enum.variant`; payload type with its own capture. Plan 02 of the grammar may have added this file already; extend it.
   - `keywords.roc`: `match`, `where`, `as`, `to`, `exposing`, `if`, `else`, `for`, `in`, `while`, `break`, `var`, `expect`, `dbg`.
   - `definitions.roc`: function definition name, annotation name for a function type, destructured names, `var` name, lambda `|`, declaration `=`.
2. Use `tree-sitter highlight <file>` to see actual captures before you write each assertion.
3. Run `tree-sitter test`.

## Out of scope

- Tests for other query files.

## Verification

- `tree-sitter test` passes, including the new highlight tests.
- Change one assertion to a wrong capture. `tree-sitter test` fails. Revert the change.

## Done when

- [ ] Five topic files exist and pass.
- [ ] `just ci` passes.
