# 04.01: Block statements with sibling annotations

Status: done (`35b9fc3d417d872f3158994e2c544dfc2b18a86c`)

Completion note: The grammar change and budget review shipped with 04.02 in the same commit. The plan allowed measured growth up to 5% after review; actual states fell from 3187 to 3142. This allowance is not the later 1% per-plan policy.

## Goal

`body_expression` accepts `annotation_type_def` as its own statement. `value_declaration` and `var_declaration` no longer contain an annotation.

## Depends on

[../03-name-fields.md](../03-name-fields.md).

## Context

- Current rules (`grammar.js:123-161`):
  - `_module_value_declaration`: `seq(decl_left, "=", body)`.
  - `value_declaration`: `seq(optional($.annotation_type_def), $._module_value_declaration)`.
  - `var_declaration`: `seq(optional($.annotation_type_def), $._module_var_declaration)`.
  - `body_expression`: `"{" repeat1(choice(value_declaration, var_declaration, local_type_binding, expect, _expr_inner)) "}"`.
- After grammar plan 03, `decl_left` has a `pattern:` field and `annotation_type_def` has a `name:` field.
- Parser budget: `tools/check-parser-metrics.js` (states 3140, large states 877, bytes 5161245). Record the metrics before the change with `just parser-metrics-check`.
- The risk: an annotation ends with a type. A type can continue with `(`, `,`, `->`, `where`. The next statement starts with an identifier. In blocks, the parser needs to know where the annotation ends. The module level has the same issue and solves it; check how (`_module_elem` repetition in `file`).

## Steps

1. Record baseline metrics: `just parser-metrics-check`.
2. Change the rules:
   - `body_expression`: replace `$.value_declaration` and `$.var_declaration` with `$.annotation_type_def`, `alias($._module_value_declaration, $.value_declaration)`, `alias($._module_var_declaration, $.var_declaration)`.
   - Delete `value_declaration` and `var_declaration` as standalone rules if nothing else uses them. Check with `grep -n 'value_declaration\|var_declaration' grammar.js`.
   - Rename `_module_value_declaration` and `_module_var_declaration` to `_value_declaration` and `_var_declaration`, because they are no longer module-specific. Update every reference.
   - Delete the comment at `grammar.js:123-124`.
3. Run `tree-sitter generate`. Then `just parser-metrics-check`.
4. If metrics exceed the budget or generation reports a new conflict, try in order. Run metrics after each attempt:
   1. Add the conflict between `annotation_type_def` and the expression that starts with the same identifier, with a comment that names the shared prefix.
   2. Give `annotation_type_def` in blocks a `prec.dynamic` lower than value declarations.
   3. End block statements with the external `_newline` token, as `_module_elem` boundaries do in `file` if they do. Read `src/scanner.c` for when the scanner emits `_newline`.
5. If an attempt passes with growth inside 5% of the baseline, raise the limits in `tools/check-parser-metrics.js` to the new values and say so in the commit message. If growth is larger, apply the stop condition in [README.md](README.md).
6. Continue with [02-corpus-and-queries.md](02-corpus-and-queries.md). Do not commit before it.

## Out of scope

- Corpus and query updates (sub-plan 02).
- roc-ts-mode changes.

## Verification

- `tree-sitter generate` succeeds.
- `just parser-metrics-check` passes.
- `tree-sitter parse` on this input shows two siblings in the block and no ERROR:
  ```roc
  f = |x| {
      y : I64
      y = x + 1
      y
  }
  ```

## Done when

- [x] No `optional($.annotation_type_def)` remains in declaration rules.
- [x] Metrics pass, or the stop condition is applied.
