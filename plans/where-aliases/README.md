# Where-alias plans

These plans make `src/build/roc/Builtin.roc` from the Roc compiler parse without `ERROR` or `MISSING` nodes. The Zed outline test uses that file as its fixture. The outline test starts after W01 and W02.

## Builtin.roc parse errors

Source: Roc `6de96ed3afba20699688bc30ce40c062267f2382` (upstream `main`, 2026-10-22), `src/build/roc/Builtin.roc`, 28,519 lines. The local checkout `~/Projects/roc` at `d93715878ca0258b24576a6649928df0dbea8f52` (branch `local-install`) has the same errors at the same lines.

The grammar at `8b02942` parses the file with 14 distinct `ERROR` regions. All regions come from 2 root causes:

| Plan | Cause | Builtin.roc lines |
|---|---|---|
| W01 | A where-alias header has type parameters: `a.Encodable(err) :` | 235, 243, 2117 |
| W02 | A where-alias reference has type arguments: `where [a.Encodable([])]` | 248, 261, 271, 288, 305, 2123, 2135 |

The `ERROR` regions that start at lines 1488, 2054, 2219, and the last `}` of the file are recovery cascades. A copy of the file with the type arguments removed from those 10 lines parses with 0 `ERROR` and 0 `MISSING` nodes.

The same forms without type arguments parse clean: `a.Encodable :` and `where [a.Encodable]`. A `where` clause on its own line does not cause errors.

## Order

| Plan | Change | Depends on | Needed for Builtin.roc |
|---|---|---|---|
| [01-parameterized-alias-header.md](01-parameterized-alias-header.md) | Type parameters on `where_alias_name` | none | Yes |
| [02-parameterized-alias-reference.md](02-parameterized-alias-reference.md) | Type arguments on `where_alias_ref` | W01 | Yes |
| [03-qualified-alias-reference.md](03-qualified-alias-reference.md) | Qualified names in `where_alias_ref` | W02 | No |

Each plan regenerates `src/parser.c`. Do the plans serially. Each plan is one commit.

## Constraints

Obey the "Constraints for every grammar plan" in [../grammar/README.md](../grammar/README.md). Two of these constraints are stop conditions:

- If `just parser-metrics-check` fails, stop and report the metrics. Do not raise the ceilings without operator approval.
- Do not add a conflict entry without a comment that states the shared prefix.

## Done when

- W01 and W02 are done.
- `./node_modules/.bin/tree-sitter parse` reports 0 `ERROR` and 0 `MISSING` nodes for `Builtin.roc` at the revision recorded above.
- `just ci` passes.
