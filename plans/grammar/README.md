# Grammar plans

Do these plans in order. Each plan is one commit or one PR. Each plan regenerates `src/parser.c`, so two plans in parallel produce conflicts.

| Plan | Change | Depends on |
|---|---|---|
| [01-try-operator.md](01-try-operator.md) | One `try_operator` node for postfix and infix `?` | none |
| [02-tag-type-name.md](02-tag-type-name.md) | Named `tag_name` node in `tag_type` | 01 |
| [03-name-fields.md](03-name-fields.md) | `name:` and `pattern:` fields on definitions | 02 |
| [04-annotation-shape/](04-annotation-shape/README.md) | Annotations are siblings in blocks, as at module level | 03 |
| [05-keyword-tokens.md](05-keyword-tokens.md) | Keywords are anonymous tokens | 04 |
| [06-supertypes.md](06-supertypes.md) | Enable `supertypes` | 05 |
| [07-expr-body-wrappers/](07-expr-body-wrappers/README.md) | Remove `expr_body` wrappers where a field names the role | 06 |

The dependencies are ordering constraints on `src/parser.c` and the corpus. They are not logical dependencies. If a plan fails, the next plan can start from the last passing commit.

## Completion

Grammar plans 01–07 are done: 01 `e4b700c`, 02 `85f333e`, 03 `7d14921`, 04 `35b9fc3`, 05 `e4f8365`, 06 `10d42dc`, 07 `c0f87e4` (with the follow-ups documented in their leaf plans). This does not mark the separate query or Emacs-mode plans done.

Completed side task — feature-preserving parser size reduction: from the post-07 scope-fix baseline of **3135 states / 905 large states / 5,221,443 bytes** (`45fb54d`) to **3133 / 903 / 5,213,834** (`a0a9287` through `5aaf06b`). `431bed1` kept explicit declaration-body query alternatives synchronized with the `_expr_inner` supertype without duplicating captures; `d84e423` inlined the infix try token and locked parser ceilings; `a0a9287` corrected the exact byte ceiling; `b797742` made the node-types guard accept either generated JSON shape. `5aaf06b` added the CI highlight-fixture completion guard so a successful CLI exit cannot silently skip later fixtures. This work changed no intended syntax, parse-tree ranges, or query capture multiplicity.

## Constraints for every grammar plan

1. Read the current rule before you edit it. Line numbers in the plans can drift.
2. After the edit, run `tree-sitter generate`. Fix new conflicts with the smallest change. Do not add a conflict entry without a comment that states the shared prefix.
3. Run `just parser-metrics-check`. Do not raise the limits in `tools/check-parser-metrics.js` unless the plan says so. If a plan exceeds the budget, stop and report the metrics.
4. Run `just test-update`. Read the full corpus diff. Each changed tree must show only the intended change.
5. Update every file in `queries/` that names a changed node: `highlights.scm`, `locals.scm`, `tags.scm`, `textobjects.scm`, `indents.scm`, `rainbows.scm`, `injections.scm`. Use `grep -n '<node_name>' queries/*.scm`.
6. Run `just queries-generate`. Add map entries in `query-maps/*.json` for new capture names.
7. Run `just ci`. It must pass.
8. Do not edit `~/Projects/roc-ts-mode`. The mode adapts in [../emacs-mode/](../emacs-mode/README.md).

## Comments in `grammar.js`

A comment states a constraint that the code cannot show. Do not add comments that narrate the change or cite plans.
