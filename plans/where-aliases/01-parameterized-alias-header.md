# W01: Type parameters on where-alias headers

Status: not started

## Goal

A where-alias header accepts type parameters: `a.Encodable(err) : where [...]`. The parameters are visible in the tree.

## Depends on

None.

## Context

- Repro:
  ```roc
  a.Enc(err) :
  	where [
  		a.enc : a -> Try(Str, err),
  	]
  ```
  The grammar at `8b02942` reports `ERROR [0, 5] - [0, 10]` for `(err)`.
- Builtin.roc sites (Roc `6de96ed`): lines 235, 243, and 2117. All 3 sites are inside `nominal_methods` blocks.
- Upstream syntax: [`parseTypeHeaderTokens`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L1113) parses where-alias headers with `name_tag = .dotted_upper_ident`.
  - [Line 1117](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L1117) accepts `NoSpaceDotUpperIdent` and `DotUpperIdent` as the name.
  - [Line 1132](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L1132) accepts only `LowerIdent`, `NamedUnderscore`, and `Underscore` as arguments. Arguments are comma-separated with an optional trailing comma.
  - The parser accepts `NoSpaceOpenRound` and `OpenRound`. A space before `(` is legal upstream.
- Upstream snapshots without parse problems: [`where_alias_parameterized`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_parameterized.md), [`where_alias_parameter_constraint`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_parameter_constraint.md), [`where_alias_arity_mismatch`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_arity_mismatch.md).
- Current rules in `grammar.js`:
  - `alias_type_def` has a second alternative: `seq(field("name", $.where_alias_name), ":", "where", field("body", $.static_dispatch_list))`.
  - `where_alias_name` is `bound_variable`, immediate `.`, then an immediate uppercase token aliased to `concrete_type`. It has no argument list.
  - Ordinary type headers use `apply_type`, which is `concrete_type` plus optional `_apply_type_args`. `_apply_type_args` uses an immediate `(` and the field `type_args`.
- No file in `queries/` or `test/highlight/` names `where_alias_name`.
- Corpus: `test/corpus/where_aliases.txt` has 6 cases.

## Steps

1. Add `optional($._apply_type_args)` to the end of `where_alias_name`. This gives the same `type_args: (apply_type_arg ...)` shape as ordinary type headers.
2. Run `tree-sitter generate`. If the change makes a conflict, fix it with the smallest change. If you cannot fix it without a conflict entry, report the conflict.
3. Run `just parser-metrics-check`. If it fails, stop and report the metrics.
4. Add corpus cases to `test/corpus/where_aliases.txt`:
   - 1 parameter: `a.Encodable(fmt) : where [a.encode : a, fmt -> fmt]`.
   - 2 parameters with a trailing comma.
   - A multiline `where` body, as in Builtin.roc line 235.
   - A parameterized header inside a `nominal_methods` block.
   - An underscore parameter.
5. Run `just test-update`. Read the full corpus diff. Only the new cases change.
6. Check the highlight of the new argument nodes with `tree-sitter highlight`. If `bound_variable` inside `type_args` gets no type-variable capture, add a highlight assertion and a query change.
7. Run `just queries-generate` and `just ci`.

## Out of scope

- Type arguments on where-alias references. W02 does this.
- Qualified where-alias names. W03 does this.
- A space before `(` and a space before `.`. Ordinary type headers use an immediate `(` too. Upstream accepts the spaced forms. This divergence is common to all type headers, so this plan does not change it.
- Rejection of non-variable arguments. `_apply_type_args` accepts any type. Ordinary type headers accept any type too. The compiler reports `invalid_type_arg`, and the grammar does not.

## Verification

- The repro parses with 0 `ERROR` nodes.
- `Builtin.roc` lines 235, 243, and 2117 are not in `ERROR` regions. W02 errors can remain.
- `just ci` passes.

## Done when

- The corpus cases pass.
- `just ci` passes within the parser budget.
- The commit includes the regenerated `src/parser.c`, `src/grammar.json`, and `src/node-types.json`.
