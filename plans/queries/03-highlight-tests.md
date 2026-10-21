# Q03: Highlight tests for base captures

Status: complete (2026-10-21). All five topic fixtures pass; no grammar or query changes were needed.

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

- [x] Five topic files exist and pass.
- [x] `just ci` passes.

## Implementation and verification

- Added `try_operator.roc` (4 assertions), `crash.roc` (1), `keywords.roc` (14), and `definitions.roc` (13). Extended `tag_types.roc` to 6 assertions, including a user-defined payload type. Tag names assert the current full base capture, `type.enum.variant.roc-special.builtin`.
- Inspected standalone HTML highlighting and query captures before adding assertions. Standalone highlighting used a temporary CLI parser-directory configuration; no global settings changed.
- `npm run ts:test` passed: 198 corpus parses and all seven highlight fixtures completed.
- Temporarily changed the `crash` assertion to `keyword.control.return`. The local `tree-sitter test` exited 1 and reported actual capture `keyword.control.exception`; restored the correct assertion.
- Final `just ci` passed, including all seven highlight fixtures, generated-artifact checks, parser budgets, generated-query checks, exact-capture regressions, and 17 shared-capture checks. This verifies CLI captures, not editor rendering.
