# 01: One node for postfix and infix `?`

Status: done (`e4b700c7d30cbb1a61774eb2a940e2151b8c52e5`; guard follow-ups `536a7120007c739abdc99fd0a9f6847cb2d55e09`, `1771e883a59f6aa1a125d9144520bdface639c9f`)

Completion note: Both forms produce `try_operator`; `??` remains an operator. The negative `(operator "?")` pattern is rejected as an impossible query, rather than compiling with zero captures. Highlighting uses a tagged capture list, `[(try_operator) "return"] @keyword.control.return`, not a standalone capture. The follow-ups standardize generated node types and restore Neovim highlighting.

## Goal

Postfix `?` and infix `?` produce the same named node, `try_operator`. One query pattern then matches both forms in every editor.

## Depends on

None.

## Context

- Postfix `f(x)?` parses as `(suffix_op_expr (function_call_pnc_expr ...) (suffix_operator))`. Rules: `suffix_op_expr` (`grammar.js:340-341`), `suffix_operator` and `suffix_operator_identifier` (`grammar.js:920-921`).
- Infix `f(x) ? handler` parses as `(bin_op_expr ... (operator) ...)`. The `"?"` choice is in `_non_pipe_operator_identifier` (`grammar.js:950`).
- `??` is `token(prec(1, "??"))` in the same choice (`grammar.js:949`). It stays an `(operator)`.
- The query `"?"` matches only the infix form. Editors need two patterns plus a text check today.
- Corpus tests: `test/corpus/modern_syntax.txt`, test "spaced try handler and immediate suffix" (about line 937) and the suffix tests near line 1539. `test/corpus/examples.txt:109-110`.
- `queries/highlights.scm:185` and `:195` capture `(suffix_operator)`.

## Steps

1. Add a rule `try_operator` that produces a named leaf for `?`. Postfix `?` must stay an immediate token (no space before it). Infix `?` has a space before it. Two token definitions with one alias is acceptable:
   - Postfix: `alias(token.immediate("?"), $.try_operator)`.
   - Infix: `alias("?", $.try_operator)`.
2. Replace `$.suffix_operator` in `suffix_op_expr` with the postfix form. Delete `suffix_operator` and `suffix_operator_identifier`. Remove `$.suffix_operator` from `inline` (`grammar.js:97`).
3. Remove `"?"` from `_non_pipe_operator_identifier`.
4. Change `bin_op_expr` (`grammar.js:325-337`) so the repeated operator position is `choice($.operator, <infix try_operator>)`.
5. Keep the node name `suffix_op_expr`. Renaming it is out of scope.
6. Run `tree-sitter generate`. If a conflict appears between postfix and infix `?`, the immediate token decides; check that `a? b` and `a ? b` both parse as intended.
7. Run `just test-update`. Expected diff: `(suffix_operator)` becomes `(try_operator)`. Infix `(operator)` before a handler becomes `(try_operator)`. No other changes.
8. Add corpus cases to `test/corpus/operators.txt`:
   - `x = a ? |e| E(e) |> f` (infix try then pipe in one flat `bin_op_expr`).
   - `x = a?.b` only if the grammar accepts field access after `?`; otherwise skip.
   - `x = a ?? b ? c` to confirm `??` stays `(operator)`.
9. Update `queries/highlights.scm`:
   - Replace both `(suffix_operator)` captures with `(try_operator)`.
   - Use one capture: `(try_operator) @keyword.control.return`. Remove `(suffix_operator)` from the `@keyword.operator` list.
10. Run `just queries-generate` and `just ci`.

## Operator-as-function compatibility

The review on 2026-10-21 confirmed that `value = (?)` is rejected after this change. Keep that rejection: current Roc treats operators as syntax rather than function values. Evidence: [current parser at 37daf709](https://github.com/roc-lang/roc/blob/37daf70979eae0cbf6bc603ab9b6721076ed0654/src/parse/Parser.zig) and [official operator documentation](https://github.com/roc-lang/roc/blob/b8040bf6a224cc82a0d146a960c3f8dfca20d58b/roc-for-elm-programmers.md). The older Tree-sitter grammar accepted `(?)`; that is not evidence that an older compiler accepted it. `test/corpus/try_operator_value.txt` locks the rejection. Other legacy operator-as-function forms are unchanged by this review.

## Out of scope

- A separate `try_expr` node with a `handler:` field.
- Changes in roc-ts-mode.

## Verification

- `just ci` passes.
- `tree-sitter query` with `(try_operator) @t` on a file containing `a = f(x)?` and `b = f(x) ? |e| E(e)` returns two captures.
- `(operator "?")` returns no captures.

## Done when

- [x] No rule named `suffix_operator` exists.
- [x] Corpus shows `(try_operator)` for both forms.
- [x] `queries/highlights.scm` has no `suffix_operator`.
- [x] `just ci` passes.
