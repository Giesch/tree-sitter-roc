# 06: Supertypes

Status: done (`10d42dc772f5ba8e868307caae7f73724becaa4b`)

Completion note: The four generated supertypes are `_expr_inner`, `_pattern`, `_atomic_type`, and `_module_elem`; no public `expression` supertype was introduced. `_atomic_type` excludes `function_type`, and the generator accepts anonymous `"*"` without a wildcard wrapper. Parse trees did not change. The measured state growth, 3120 to 3153 (+33, 1.0577%), was explicitly approved above the normal 1% per-plan gate; the normal gate remains for later plans.

## Goal

`node-types.json` declares supertypes for expressions, patterns, and types. Queries can use supertype syntax such as `(_expr_inner/anon_fun_expr)`. Typed consumers (Rust bindings, Zed, Helix text objects) get a node hierarchy. Parse trees do not change.

## Depends on

[05-keyword-tokens.md](05-keyword-tokens.md) (ordering only).

## Context

- `supertypes` is commented out at `grammar.js:102`: `// supertypes: ($) => [$._module_elem, $._pattern, $._expr_inner],`.
- Tree-sitter requires each supertype to be a hidden rule whose body is a `choice` of single symbols. Hidden members are allowed; tree-sitter flattens them in `node-types.json`.
- Candidates:
  - `_expr_inner` (`grammar.js:211`): `choice($.bin_op_expr, $._atom_expr)`. `_atom_expr` and `_postfix_expr` and `_primary_expr` are nested hidden choices.
  - `_pattern` (`grammar.js:402-420`): contains `alias("_", $.wildcard_pattern)` and `alias($.const, $.const_pattern)`. Aliases inside a supertype choice can fail; test it.
  - `_atomic_type` (`grammar.js:668-680`): contains the anonymous `"*"`. An anonymous string member is not allowed in a supertype.
  - `_module_elem` (`grammar.js:107-119`).
- The `inline` list (`grammar.js:86-100`) must not contain a supertype.

## Steps

1. Enable `supertypes: ($) => [$._expr_inner, $._pattern, $._atomic_type, $._module_elem]`.
2. Run `tree-sitter generate`. For each error:
   - Anonymous string member (`"*"` in `_atomic_type`): wrap it in a named rule, for example `wildcard_type: () => "*"`. This changes the tree for `*`; record it.
   - Alias member: move the alias into a separate named rule if generation rejects it.
   - Drop a candidate from the list if the fix needs more than a local change. Record the reason in the commit message.
3. Run `just parser-metrics-check`.
4. Run `just test`. Expected: no corpus changes, except the `wildcard_type` change if step 2 added it. Run `just test-update` only for that case.
5. Inspect `src/node-types.json`. Each enabled supertype has `"subtypes"`.
6. Supertype names in queries keep the leading underscore, for example `(_expr_inner/anon_fun_expr)`. If a public name such as `expression` is wanted, rename the hidden rule (`_expr_inner` to `_expression`) only if the rename stays inside `grammar.js`. Hidden rule names do not appear in trees.
7. Do not rewrite existing queries to use supertypes in this plan.
8. Run `just ci`.

## Out of scope

- Rewriting queries.
- Removing `expr_body`. That is plan 07.

## Verification

- `just ci` passes.
- `jq '[.[] | select(.subtypes) | .type]' src/node-types.json` lists the enabled supertypes.
- A query using a supertype compiles: `(_expr_inner/anon_fun_expr) @f` (adjust to the final name).

## Done when

- [x] At least `_expr_inner` and `_pattern` are supertypes, or the commit message records why not.
- [x] Corpus is unchanged except recorded cases.
- [x] `just ci` passes.
