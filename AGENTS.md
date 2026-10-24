# tree-sitter-roc

Last verified: 2026-10-23

## Workflow
- JavaScript grammar with a Tree-sitter external scanner; `grammar.js` is the grammar source, `src/scanner.c` handles scanner behavior.
- Use the project-local Tree-sitter CLI pinned in `package.json` (currently 0.25.10). npm scripts resolve it automatically; direct commands use `./node_modules/.bin/tree-sitter`.
- `npm run ci` checks generated artifacts, parser-size budgets, corpus inventory, parser/highlight tests, generated queries, expression-body captures, shared highlights, operator full-tree regressions with comparator self-tests, and the Zed outline snapshot with production-query self-tests.
- `node tools/check-operator-trees.js --self-test` compares source and linewise XML, including operator fields, anonymous tokens, ranges, trivia, and mid-file recovery. The pinned CLI omits explicit missing-anonymous-token metadata from XML; the guard also checks its recovery diagnostic.
- Operator baseline refreshes require a reviewed syntax-tree contract change. Record from the approved parser with production grammar, scanner, and generated files matching HEAD. `--record` creates a baseline; replacing an existing baseline requires explicit `--record --force`. Review the source/XML diff; never refresh it to hide a candidate regression. Normal verification does not write the baseline.
- `npm run ts:generate-check` regenerates `src/parser.c`, `src/grammar.json`, and `src/node-types.json`, checks their format/version, and requires a clean generated diff. Commit generated artifacts with grammar changes.
- `npm run ts:test` runs the corpus inventory guard and the test wrapper that verifies every highlight fixture completed. A bare CLI success can miss later highlight assertions; use the wrapper for verification.
- `npm run queries:generate` regenerates editor queries. See `queries/AGENTS.md` for query contracts.
- Parser growth is bounded by `tools/check-parser-metrics.js`: 3149 states, 905 large states, and 5234140 parser bytes. The operator approved exact measured increases for parameterized where-alias headers and references within a cumulative 2% of the original pre-W01 ceilings (3144 states / 905 large states / 5231479 bytes) across W01–W03, not per fix. Current cumulative growth is +5 states / +0 large states / +2661 bytes; no speculative headroom is included. The scoped exception and source evidence are recorded in `plans/where-aliases/03-qualified-alias-reference.md`. Stop if any metric exceeds its original ceiling times 1.02.

## Structure
- `grammar.js`, `src/scanner.c`: editable language implementation; `src/grammar.json`, `src/node-types.json`, and `src/parser.c`: generated outputs.
- `test/corpus/`, `test/highlight/`, `test/fixtures/`: syntax trees, highlight assertions, and query-regression inputs.
- `queries/`, `query-maps/`, `queries-generated/`: base queries, editor capture mappings, and generated editor outputs.
- `tools/`: generation and regression guards; `bindings/`: language bindings.

## Syntax-tree contracts
- `src/node-types.json` is a standard top-level array; the parser generator banner must match the pinned CLI version.
- `_expr_inner`, `_pattern`, `_atomic_type`, and `_module_elem` are supertypes. Consumers should use fields and concrete children rather than assume hidden wrapper nodes exist.
- `value_declaration` exposes `pattern: (decl_left)` and a direct expression `body`; `var_declaration` exposes `name: (identifier)` and a direct expression `body`.
- `annotation_type_def` and `inferred_associated_annotation` expose `name: (annotation_pre_colon)`; ordinary alias, opaque, and nominal type definitions expose `name: (apply_type)`.
- Constraint aliases such as `item.Eq : where [...]` use `alias_type_def` with `name: (where_alias_name)` and `body: (static_dispatch_list)`. The name contains `bound_variable` and `concrete_type` leaves, plus optional `type_args: (apply_type_arg ...)` for contiguous header parameters such as `a.Encodable(err)`. Arguments reuse ordinary type application syntax (any type, with an optional trailing comma); compiler validation restricts header parameters. Constraint function types reuse inline-where nodes and captures. Lists can mix inline `static_dispatch` entries with `where_alias_ref` references. References accept contiguous type arguments through optional repeated `type_args: (apply_type_arg ...)`, including general types and an optional trailing comma; whitespace before `(` ends the reference. References have a lowercase receiver followed by one or more uppercase segments, each retained as a separate `concrete_type` leaf. Space, tab, LF, CRLF, and comments may precede each dot; the dot must touch the uppercase segment. Their `bound_variable`, `concrete_type`, and argument leaves reuse existing highlights.
- Block annotations are sibling `annotation_type_def` statements, not children of the following declaration. Match an annotation/definition pair by sibling position and name where appropriate.
- `match_branch` exposes `pattern`, optional `guard`, and direct expression `expr`. Fielded expression bodies no longer introduce `expr_body` wrappers; wrappers still exist in other contexts.
- Both postfix and infix `?` expose `try_operator`; postfix `?` is immediate. Keywords such as `as`, `where`, and `to` are anonymous tokens, not named keyword nodes.
- `tag_type` exposes `name: (tag_name)`; import exposing lists use `exposing: (exposes_list)` rather than an `exposing` node alias.
- Module exports and import exposures retain `ident` for plain names. Richer items use `qualified_exposed_name` (`root`, repeated `segment`), `renamed_exposed_item` (`name`, `alias`), and `wildcard_exposed_item` (`name`, anonymous `wildcard: ".*"`). Qualified segments are contiguous; trivia is permitted before `.*`. Syntax follows the versioned parser evidence in `plans/side-findings/module-exposure-forms.md`, not semantic name resolution.
