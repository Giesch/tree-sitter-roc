# 05: Keywords as anonymous tokens

Status: done (`e4f8365c72bb29c65b3da0b665a9574eb078f361`; guard fixture `c61b4766469d0cbc2ad6810b77f1959bf35e97e9`)

Completion note: The anonymous keywords, `exposing:` field, and `guard:` field are implemented; the follow-up corpus test covers the guard field. The corpus changes include ambiguity handling, not solely removal of alias nodes. Parser growth for this plan falls under the later 1% per-plan approval policy.

## Goal

Keywords are anonymous tokens everywhere. Named aliases remain only for nodes that carry structure. Every editor's keyword list is then a flat list of strings.

## Depends on

[04-annotation-shape/](04-annotation-shape/README.md) (ordering only).

## Context

| Location | Current | Target |
|---|---|---|
| `_match_start` (`grammar.js:343`) | `alias("match", $.match)` | `"match"` |
| `where_implements` (`grammar.js:704`) | `alias("where", $.where)` | `"where"` |
| `_import_body` (`grammar.js:550`) | `alias("as", $.as)` | `"as"` |
| `import_file_expr` (`grammar.js:557`) | `alias("as", $.as)` | `"as"` |
| `to` rule (`grammar.js:560`), used at `:571` | named node `(to)` | `"to"`; delete the rule |
| `_import_body` (`grammar.js:551`) | `alias($.exposes_list, $.exposing)` | `field("exposing", $.exposes_list)` |
| `match_branch` (`grammar.js:350`) | `alias($._expr_inner, $.if)` | `field("guard", $._expr_inner)` |

- The `(exposing)` alias names the whole `exposing [a, b]` list, including brackets and identifiers. roc-ts-mode highlights the whole list as a keyword.
- The `(if)` alias in `match_branch` names the guard expression, not the keyword.
- Keep these aliases. They are operators or values, not keywords: `double_colon`, `colon_equals`, `optional_field`, `default_value`, `optional_access`, `unset_record_field`, `wildcard_pattern`.
- `break_expr` stays. It is an expression node.
- Query references: `queries/highlights.scm:120` (`exposing`), `:213` (`match`), `:218` (`as`), `:233` (`to`), `:235` (`where`). `queries/locals.scm:23` (commented `(as)`).

## Steps

1. Make each change in the table.
2. Run `tree-sitter generate` and `just parser-metrics-check`.
3. Run `just test-update`. Expected diff:
   - `(match)`, `(where)`, `(as)`, `(to)` disappear from trees.
   - `(exposing ...)` becomes `exposing: (exposes_list ...)`.
   - `(if ...)` in match branches becomes `guard: <expression>`.
4. Update `queries/highlights.scm`:
   - Replace `(match)`, `(as)`, `(to)`, `(where)` with `"match"`, `"as"`, `"to"`, `"where"`.
   - Change line 120 to `(import_expr exposing: (exposes_list (ident) @special.roc-special.exposed))`.
   - Confirm `"exposing"` is in the keyword list (it is, at line 223).
5. Grep all `queries/*.scm` for the removed node names and fix each match.
6. Run `just queries-generate` and `just ci`.

## Out of scope

- Changing highlight groups for keywords.

## Verification

- `just ci` passes.
- `grep -nE '\((match|where|as|to|exposing|if)\)' queries/*.scm` returns nothing.
- `tree-sitter query` with `["match" "where" "as" "to"] @k` on a file that uses each keyword returns one capture per keyword.

## Done when

- [x] No named keyword aliases remain in `grammar.js` from the table.
- [x] Queries use string tokens.
- [x] `just ci` passes.
