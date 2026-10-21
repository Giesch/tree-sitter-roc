# 03: Fields for definition names

Status: done (`7d14921931ec346a945d40b8073e61612ec713cc`)

Completion note: Definition fields and type-definition tags were added; the corpus diff contains field labels only.

## Goal

Every definition node exposes its name or binding pattern through a field. Consumers find names with `treesit-node-child-by-field-name` or a field query, not with positional child access.

## Depends on

[02-tag-type-name.md](02-tag-type-name.md) (ordering only).

## Context

| Node | Rule | Field to add |
|---|---|---|
| `annotation_type_def` | `grammar.js:628` | `name:` on `annotation_pre_colon` |
| `inferred_associated_annotation` | `grammar.js:662-663` | `name:` on `annotation_pre_colon` |
| `value_declaration` / module form | `grammar.js:125-132` | `pattern:` on `decl_left` |
| `alias_type_def` | `grammar.js:629` | `name:` on `apply_type` |
| `opaque_type_def` | `grammar.js:631-637` | `name:` on `apply_type` |
| `nominal_type_def` | `grammar.js:641-647` | `name:` on `apply_type` |
| `var_declaration` | `grammar.js:134-142` | has `name:` already |

- `annotation_pre_colon` (`grammar.js:785-791`) is a wrapper whose only child is `identifier`. It holds commented-out alternatives.
- roc-ts-mode reaches names with chains like `(treesit-node-child (treesit-node-child node 0 t) 0 t)`. The mode adapts later in `emacs-mode/03`.
- Queries that name `annotation_pre_colon` or `decl_left`: `queries/highlights.scm:38`, `:248-256`, `:267-273`, `:395-398`; `queries/tags.scm:10`.

## Decision

Keep the `annotation_pre_colon` and `decl_left` nodes. Add fields on the parent. Folding `annotation_pre_colon` into a direct `name: (identifier)` field changes every annotation in the corpus and every query that names it; this plan does not do that.

## Steps

1. Wrap each child in the table with `field(...)`. For aliased rules (`alias($._assignment_pattern, $.decl_left)`), put `field("pattern", ...)` around the alias.
2. Run `tree-sitter generate` and `just parser-metrics-check`. Fields do not add states; a change in state count means a rule changed by mistake.
3. Run `just test-update`. Expected diff: field labels only (`name:`, `pattern:`). No node type changes.
4. Update queries to use the fields where a field removes a positional assumption. Example: `(annotation_type_def name: (annotation_pre_colon (identifier) @function) (function_type))`.
5. Update `queries/tags.scm` with definitions for types: `(alias_type_def name: (apply_type (concrete_type) @name)) @definition.type`, and the same for `opaque_type_def` and `nominal_type_def`.
6. Run `just queries-generate` and `just ci`.

## Out of scope

- Moving annotations out of block declarations. That is plan 04.
- Removing `annotation_pre_colon` or `decl_left`.

## Verification

- `just ci` passes.
- `tree-sitter tags` on a file with `T : [A]`, `N := [B]`, and `f = |x| x` lists `T`, `N`, and `f`. If `tree-sitter tags` needs parser-directory configuration, use `tree-sitter query queries/tags.scm <file>` instead.

## Done when

- [x] Every node in the table has its field.
- [x] Corpus diff contains field labels only.
- [x] `queries/tags.scm` has type definitions.
- [x] `just ci` passes.
