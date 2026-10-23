# W02: Type arguments on where-alias references

Status: done (W02 commit containing this record)

## Execution evidence — 2026-10-23

- Started at W01 commit `3f0031cf143c5ad3f18bc7c4191cc532d994e55d`. Added only `optional($._apply_type_args)` to `where_alias_ref`; no scanner, conflict, or qualified-reference changes.
- Generated parser: **3146 states / 905 large states / 5232408 bytes**. W02 adds 1 state and 500 bytes to W01. The initial metrics guard failed against W01 ceilings; the operator explicitly authorized exact measured increases within a cumulative 2% of original pre-W01 ceilings **3144 / 905 / 5231479** across W01–W03, not per fix. Current cumulative growth is **+2 / +0 / +929** (0.063614% / 0% / 0.017758%). Only states and bytes ceilings were raised, to these measurements; stop above any original ceiling times 1.02.
- Ten added corpus cases cover empty unions, variables, open unions, own-line `where`, both inline-constraint orders, multiple arguments/trailing comma, spacing and empty-argument recovery, and the exact SOURCE sections of all four named upstream snapshots at `6de96ed3afba20699688bc30ce40c062267f2382`. New valid trees retain `type_args` fields; all pre-existing corpus content is byte-identical.
- `tree-sitter test --update --show-fields` generated the valid expectations. Its broad formatting updates were removed; malformed-case expectations were inspected and entered explicitly. Guarded tests pass: **247 corpus parses**, **all 14 highlight fixtures**, including 10 reference-fixture assertions. Existing leaf queries cover reference arguments; no query change is needed.
- `Builtin.roc` was extracted with `git show` from the pinned revision, not read from the current checkout. Expected and extracted blob hashes both equal `eb0de2678ce94ee549385770842b767f6ff16c59`. Parse exit 0 with **0 ERROR / 0 MISSING**.
- A second pinned parser generation is byte-stable for all three generated artifacts (SHA-256 checked before/after). Standalone `tree-sitter highlight --html --css-classes` succeeds with a temporary parser-directory config; argument leaves have existing type captures. `git diff --check` passes.
- All substantive npm CI checks pass: node-types self-tests, parser metrics, corpus inventory, guarded corpus/highlight tests, generated-query checks, expression-body exact captures, shared highlights, and operator full-tree regressions/self-tests. Queries regenerate unchanged.
- Requested independent reviews completed with no unresolved Critical or Important findings. The two new malformed-case expectations were expanded to one node per line without changing fields or recovery; pinned Builtin parsing and regeneration stability were independently confirmed.
- Literal `npm run ci` passed with exit 0 after staging the eleven intended tracked paths, including generated-artifact and generated-query clean-diff gates, 247/247 corpus parses, all 14 highlight fixtures, exact-capture checks (locals 189 / indents 69 / textobjects 41 / rainbows 139 / highlights 512 / tags 16), 18 shared captures, and operator regression/self-tests (22 operators / 5 recovery fixtures). The earlier unstaged clean-diff failure is resolved. W03 remains out of scope.

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
