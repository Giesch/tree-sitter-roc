# W02: Type arguments on where-alias references

Status: not started

## Goal

A where-alias reference in a `where` list accepts type arguments: `where [a.Parseable([InvalidJson(Str), ..errs])]`. The arguments are visible in the tree.

## Depends on

[01-parameterized-alias-header.md](01-parameterized-alias-header.md). Both plans edit the where-alias rules and regenerate the parser.

## Context

- Repro:
  ```roc
  to_str : a -> Str where [a.Enc([])]
  parse : Str -> Try(a, errs)
  	where [a.Parse(errs)]
  ```
  The grammar at `8b02942` reports `ERROR` regions for the `([])` and `(errs)` arguments.
- Builtin.roc sites (Roc `6de96ed`): lines 248, 261, 271, 288, 305, 2123, and 2135. The arguments include `[]`, `err`, `[InvalidJson(Str), ..errs]`, and `[BadHeader, ..errs]`.
- Upstream syntax: the [where-alias reference parser](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L5957) reads the name with [`readWhereAliasQualificationChain`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L3332). Then, at [line 5971](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L5971), a `NoSpaceOpenRound` starts general type-apply arguments. A space before `(` ends the reference.
- Upstream snapshots without parse problems: [`where_alias_parameterized`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_parameterized.md), [`where_alias_argument_introduces_var`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_argument_introduces_var.md).
- Current rule in `grammar.js`: `where_alias_ref` is `bound_variable`, `.`, then an immediate uppercase token aliased to `concrete_type`. It has no argument list. `_static_dispatch_entries` chooses between `static_dispatch` and `where_alias_ref`.
- No file in `queries/` or `test/highlight/` names `where_alias_ref`.
- Corpus: `test/corpus/constraint_alias_references.txt` has 16 cases.

## Steps

1. Add `optional($._apply_type_args)` to the end of `where_alias_ref`. `_apply_type_args` uses an immediate `(`, which agrees with the upstream `NoSpaceOpenRound` rule.
2. Run `tree-sitter generate` and `just parser-metrics-check`. If the metrics check fails, stop and report the metrics.
3. Add corpus cases to `test/corpus/constraint_alias_references.txt`:
   - An empty tag-union argument: `where [a.Encodable([])]`.
   - A bound-variable argument: `where [a.Encodable(err)]`.
   - An open tag union with a spread: `where [a.Parseable([InvalidJson(Str), ..errs])]`.
   - A parameterized reference mixed with `static_dispatch` entries, in both orders.
   - A `where` clause on its own line, as in Builtin.roc line 271.
   - The source of each of the 4 parameterized upstream snapshots: `where_alias_parameterized`, `where_alias_argument_introduces_var`, `where_alias_parameter_constraint`, and `where_alias_arity_mismatch`.
4. Run `just test-update`. Read the full corpus diff. Only the new cases change.
5. Check the highlight of the new argument nodes with `tree-sitter highlight`. If they have no type capture, add a highlight assertion and a query change.
6. Run `just queries-generate` and `just ci`.

## Out of scope

- Qualified references such as `a.Holder.Show`. W03 does this.

## Verification

- The repro parses with 0 `ERROR` nodes.
- `Builtin.roc` at Roc `6de96ed` parses with 0 `ERROR` and 0 `MISSING` nodes:
  ```sh
  ./node_modules/.bin/tree-sitter parse <Builtin.roc> | grep -cE 'ERROR|MISSING'
  ```
  The expected count is 0.
- The 4 parameterized upstream snapshot sources parse with 0 `ERROR` nodes.
- `just ci` passes.

## Done when

- `Builtin.roc` parses clean.
- `just ci` passes within the parser budget.
- The commit includes the regenerated `src/parser.c`, `src/grammar.json`, and `src/node-types.json`.
- [README.md](README.md) and [../README.md](../README.md) record the status.
