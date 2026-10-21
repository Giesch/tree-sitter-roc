# Shared context

Facts that more than one plan needs. Line numbers refer to the state of the repositories when these plans were written. Confirm each reference before you edit.

## Grammar facts (`grammar.js`)

- `_expr_inner` is `choice($.bin_op_expr, $._atom_expr)`. `bin_op_expr` is a flat sequence: operand, operator, operand, and so on. It has no precedence tree.
- Postfix `?`: `suffix_op_expr` (`grammar.js:340-341`) contains `(suffix_operator)`. `suffix_operator` is a leaf with no anonymous child (`grammar.js:920-921`).
- Infix `?` (`expr ? handler`) and `??` are `(operator)` nodes inside `bin_op_expr`. `(operator)` has an anonymous child with the operator text (`grammar.js:923-951`).
- `crash_expr` (`grammar.js:233`) has an anonymous `"crash"` token and a `message:` field.
- `tag_type` (`grammar.js:723`) puts `field("name", ...)` on the hidden token `_upper_identifier`.
- Module-level annotations are siblings of their declarations (`_module_elem`, `grammar.js:107-119`). In blocks, `value_declaration` contains an optional `annotation_type_def` (`grammar.js:132`, `:142`, `:148-161`).
- Named keyword aliases: `(match)` (`grammar.js:343`), `(where)` (`:704`), `(as)` (`:550`, `:557`), `(to)` (`:560`). `(exposing)` is an alias for the whole `exposes_list` (`:551`). `match_branch` aliases its guard expression to a node named `if` (`:350`).
- `supertypes` is commented out (`grammar.js:102`).
- `expr_body` and `expr_body_terminal` are wrappers around `_expr_inner` (`grammar.js:165-166`).

## Verified query results

These results come from `tree-sitter query` against the current parser:

| Query | Result |
|---|---|
| `"?"` | Matches infix `?` only. Does not match postfix `?`. |
| `(operator "?")` | Matches infix `?`. |
| `(crash_expr "crash")` | Matches. |
| `(tag_type name: _)` | Compile error: "Impossible pattern". |
| `"else"`, `"if"` | Match both parts of `else if`. |

To run a query: write it to a file in a scratch directory, then run `tree-sitter query <query-file> <roc-file>`. Ignore the "no parser directories" warning.

## roc-ts-mode facts (`~/Projects/roc-ts-mode/roc-ts-mode.el`)

- Font-lock rules are hand-written in `roc-ts--ts-font-lock-rules` (`:207-421`). The mode does not read `queries/highlights.scm`.
- Feature list: `treesit-font-lock-feature-list` (`:617-621`).
- Elisp workarounds for grammar shape:
  - `roc-ts--fontify-question-mark` (`:199-205`) checks operator text for `?`.
  - `roc-ts--fontify-tag-type` (`:189-197`) computes the tag-name range from child positions.
  - `roc-ts--fontify-inferred-associated-annotation` (`:163-187`) walks siblings with nested first-child calls.
  - `roc-ts--ts-conditional-keyword-rules` (`:423-448`) tests whether the grammar has `while`, `break`, and `crash`. [side-findings/remove-grammar-compat.md](side-findings/remove-grammar-compat.md) removes it.
  - `roc-ts--ts-defun-name` (`:595-605`) finds names with first-child calls.
- Custom faces: `roc-ts-crash-face` (`:48-51`), `roc-ts-question-mark-face` (`:53-56`).
- Emacs in this environment is 30.1.
- The mode supports only its pinned grammar tag. See "Compatibility policy" in [README.md](README.md).

## Query generator facts (`tools/template-queries.js`)

- Every capture name in a base query file must have an entry in each `query-maps/<editor>.json`. A missing entry is an error.
- A map value of `""`, `null`, or `false` removes the capture from the generated pattern.
- Base queries put lower-priority patterns first and higher-priority patterns last. A later match wins.
