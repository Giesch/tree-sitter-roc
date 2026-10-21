# 04: Annotations are siblings in blocks

Status: done (`35b9fc3d417d872f3158994e2c544dfc2b18a86c`)

Completion note: Both sub-plans landed together. The block uses a separate `count : I64` statement before `var count = 0`; the proposed inline `var count : I64` syntax was not added. The plan allowed a measured increase up to 5%, but this commit reduced states from 3187 to 3142; the later 1% per-plan gate is separate.

## Goal

A type annotation inside a block is a sibling of its declaration, as at module level. Every consumer then handles one shape:

```
(annotation_type_def ...)
(value_declaration ...)
```

## Context

- Module level: `_module_elem` (`grammar.js:107-119`) lists `annotation_type_def` and `alias($._module_value_declaration, $.value_declaration)` as separate choices.
- Blocks: `body_expression` (`grammar.js:148-161`) lists `value_declaration` and `var_declaration`. Those rules (`grammar.js:132`, `:142`) start with `optional($.annotation_type_def)`.
- The grammar comment at `grammar.js:123-124` states the reason: separate block statements need statement boundaries, and the first attempts added GLR states.
- Consumers that depend on the two shapes:
  - roc-ts-mode indent rule for `annotation_type_def` under `value_declaration` (`roc-ts-mode.el:467`).
  - roc-ts-mode derive check, which walks siblings (`roc-ts-mode.el:163-187`).
  - `queries/highlights.scm:388-398`, which uses sibling anchors in `nominal_methods`.

## Sub-plans

| Plan | Change | Depends on |
|---|---|---|
| [01-block-statements.md](01-block-statements.md) | Grammar change and parser budget | grammar plan 03 |
| [02-corpus-and-queries.md](02-corpus-and-queries.md) | Corpus review, new tests, query updates | 01 |

Do both sub-plans in one commit. Sub-plan 01 leaves the corpus failing; sub-plan 02 fixes it.

## Stop condition

If sub-plan 01 cannot meet the parser budget after the listed attempts, stop. Revert `grammar.js`. Report the state counts for each attempt. Do not start plan 05 until the user decides.
