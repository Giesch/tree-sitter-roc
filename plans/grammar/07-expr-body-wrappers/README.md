# 07: Remove `expr_body` wrappers

Status: done (`c0f87e4bbb94555bb1cd810934c75c11f165c334`; scope follow-up `45fb54d5b4111e94e2a939fca0aa45abf749c47d`)

Completion note: Both sub-plans landed together; the follow-up restores local scopes across declaration-body expression forms. Unfielded wrappers and some parser-separation wrapper positions remain. Query capture regression checks compare names, ranges, and multiplicity, not editor-side name resolution, indentation, or rendering. Scope additions for match branches and blocks are explicit approved additions. Parser growth remains subject to the normal 1% per-plan policy.

## Goal

Where a field already names an expression's role, the expression is a direct child, without an `expr_body` wrapper. Queries become shorter: `body: (anon_fun_expr)` instead of `body: (expr_body (anon_fun_expr))`.

## Context

- `expr_body: ($) => $._expr_inner` and `expr_body_terminal: ($) => $._expr_inner` (`grammar.js:165-166`).
- `expr_body_terminal` is always aliased to `expr_body`. It exists for parser state separation.
- Uses with a field: `expect` (`body`), module declarations (`body`), `crash_expr` (`message`), `early_return_expr` (`body`), `parenthesized_expr` (`expression`), `match_branch` (`expr`), `anon_fun_expr` (`body`).
- Uses without a field: `dbg_expr`, `record_field_expr`, `record_field_type` default values, `_module_elem` top-level expressions.
- Queries that name `expr_body`: `highlights.scm:248-256`, `locals.scm:1`, `tags.scm:14`, `textobjects.scm:2`, `:16`, `:33`, `indents.scm:6`, `rainbows.scm:3`.
- `locals.scm` uses `(expr_body) @local.scope`, and `indents.scm` uses `(expr_body) @indent`. Removing the wrapper removes those scopes. Replacement patterns are needed.

## Gate

This plan touches more queries than any other. Stop and report, without committing, if any of these is true:
- Parser metrics grow more than 5% over the plan 06 baseline.
- Generation needs a new conflict entry.
- `locals.scm` or `indents.scm` cannot keep equivalent coverage.

## Sub-plans

| Plan | Change | Depends on |
|---|---|---|
| [01-grammar.md](01-grammar.md) | Grammar change and budget | grammar plan 06 |
| [02-corpus-and-queries.md](02-corpus-and-queries.md) | Corpus and every query file | 01 |

Do both sub-plans in one commit.
