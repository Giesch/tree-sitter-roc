# W03: Qualified where-alias references

Status: not started

## Goal

A where-alias reference accepts a qualified alias name: `where [a.Holder.Show]` and `where [a.Str.Utf8Problem]`.

Builtin.roc does not use this form. The Zed outline work does not need this plan.

## Depends on

[02-parameterized-alias-reference.md](02-parameterized-alias-reference.md). Both plans edit `where_alias_ref`.

## Context

- Repro:
  ```roc
  Holder := {}.{
  	a.Show : where [a.to_str : a -> Str]
  }

  describe : a -> Str where [a.Holder.Show]
  ```
  The grammar at `8b02942` reports an `ERROR` region at `[4, 35]` for this source.
- Upstream syntax: [`readWhereAliasQualificationChain`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/src/parse/Parser.zig#L3332) reads 1 or more uppercase segments after the receiver. Each segment is a `NoSpaceDotUpperIdent` or a `DotUpperIdent` token.
- Upstream snapshots without parse problems. This grammar produces `ERROR` nodes for each:
  - [`where_alias_qualified_local_type`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_qualified_local_type.md): `a.Holder.Show`.
  - [`where_alias_qualified_builtin_nested_type`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_qualified_builtin_nested_type.md): `a.Str.Utf8Problem`.
  - [`where_alias_qualified_spaced_segments`](https://github.com/roc-lang/roc/blob/6de96ed3afba20699688bc30ce40c062267f2382/test/snapshots/where_alias/where_alias_qualified_spaced_segments.md): `a .Holder .Show`.
- `where_alias_ref` uses an immediate uppercase token after `.`. A spaced segment fails for this reason too.
- `concrete_type` already represents qualified names as repeated `.` and uppercase segments.

## Steps

1. Read the tokenizer rules for `DotUpperIdent` and `NoSpaceDotUpperIdent` at Roc `6de96ed`. Record which whitespace positions the tokenizer accepts: before `.`, after `.`, or both.
2. Write the accepted and rejected forms in a table in this plan, in the format of [../side-findings/module-exposure-forms.md](../side-findings/module-exposure-forms.md).
3. Change `where_alias_ref` to accept repeated uppercase segments. Keep the leaf shape compatible with `concrete_type`, if the parser accepts it without new conflicts. Support spaced segments only if step 1 shows that the tokenizer accepts them.
4. Run `tree-sitter generate` and `just parser-metrics-check`. If the metrics check fails, stop and report the metrics.
5. Add corpus cases to `test/corpus/constraint_alias_references.txt`:
   - The 3 upstream snapshot sources.
   - A qualified reference with type arguments: `a.Holder.Show(err)`.
   - The rejected forms from step 2.
6. Run `just test-update`, read the full corpus diff, then run `just queries-generate` and `just ci`.

## Out of scope

- Qualified where-alias headers. Upstream headers have 1 uppercase segment.
- Name resolution of the qualified alias.

## Verification

- The 3 upstream snapshot sources parse with 0 `ERROR` nodes.
- `Builtin.roc` still parses with 0 `ERROR` and 0 `MISSING` nodes.
- `just ci` passes.

## Done when

- The table of accepted and rejected forms has corpus coverage.
- `just ci` passes within the parser budget.
- The commit includes the regenerated `src/parser.c`, `src/grammar.json`, and `src/node-types.json`.
