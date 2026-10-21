# 07.01: Grammar without `expr_body` under fields

Status: done (`c0f87e4bbb94555bb1cd810934c75c11f165c334`)

Completion note: Fielded expressions are direct children; remaining `expr_body` and `expr_body_terminal` rules retain unfielded/parser-separation uses. The plan's 5% stop condition was respected; the later 1% per-plan growth policy remains in effect.

## Goal

Fielded expression positions hold the expression directly.

## Depends on

[../06-supertypes.md](../06-supertypes.md).

## Context

See the use list in [README.md](README.md). `expr_body_terminal` keeps declaration bodies in separate parser states; removing it can merge states and create conflicts.

## Steps

1. Record baseline metrics: `just parser-metrics-check`.
2. Change one position at a time, in this order, and run `tree-sitter generate` plus `just parser-metrics-check` after each:
   1. `anon_fun_expr` `body`.
   2. `crash_expr` `message`, `early_return_expr` `body`, `parenthesized_expr` `expression`.
   3. `match_branch` `expr`.
   4. `expect` `body`.
   5. Module declaration `body` (`expr_body_terminal`).
3. For each position, replace `$.expr_body` (or the `expr_body_terminal` alias) with `$._expr_inner`, keeping the field.
4. If one position fails the gate in [README.md](README.md), restore that position and continue with the next. Record the kept positions for the commit message.
5. Leave positions without a field unchanged.
6. If no position remains that uses `expr_body_terminal`, delete the rule.
7. Continue with [02-corpus-and-queries.md](02-corpus-and-queries.md).

## Out of scope

- Positions without fields.

## Verification

- `tree-sitter generate` succeeds with no new conflict entries.
- `just parser-metrics-check` passes.

## Done when

- [x] Each position in step 2 is either changed or recorded as kept, with the reason.
