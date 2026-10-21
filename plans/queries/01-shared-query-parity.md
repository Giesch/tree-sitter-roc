# Q01: Base queries cover the Emacs highlighting

## Goal

For every rule in roc-ts-mode's hand-written font-lock list, `queries/highlights.scm` has a pattern with an equivalent capture. Emacs can then use generated queries without losing highlighting.

## Depends on

All plans in [../grammar/](../grammar/README.md). The patterns below use the node names from those plans: `try_operator`, `tag_name`, `name:` and `pattern:` fields, string keywords, sibling annotations, and fielded bodies without `expr_body`. If a grammar plan stopped early, use the node names in the current `src/node-types.json`.

## Context

The mode rules are in `~/Projects/roc-ts-mode/roc-ts-mode.el:207-417`, grouped by feature. This table lists each mode feature, its intent, and the base capture to use. "Exists" means the base query already covers it; confirm the pattern still matches after the grammar plans.

| Mode feature | Intent | Base capture |
|---|---|---|
| comments, doc-comments | comments | exists: `@comment.line`, `@comment.block.documentation` |
| keywords | keyword tokens | exists; confirm every keyword in the mode list is present, including `"while"`, `"break"`/`(break_expr)`, `"dbg"`, `"expect"` |
| keywords (`crash`) | warning face | new: `"crash" @keyword.control.exception`; remove `"crash"` from the `@keyword.control` list |
| early-return | `?` in its own face | new: `(try_operator) @keyword.control.return.roc-special.try`; keep `"return" @keyword.control.return` |
| strings | strings and package URIs | exists |
| string-escapes | escapes and `${ }` | exists |
| types, type-variables | `concrete_type`, `inferred`, `bound_variable` | exists |
| tag-types | tag name only | exists after grammar plan 02 |
| types (nominal constructors) | `Name.(...)` tag in type face | new: `(nominal_constructor_expr (tag (identifier) @type))`, same for `nominal_constructor_pattern` |
| numbers | numeric literals | exists |
| record-field-declaration | `field_name` | exists: `@variable.other.member` |
| record-field-access | field after a variable target | exists: `(field_access_expr (identifier) @variable.other.member)` |
| variable-use | identifiers | exists: `(identifier) @variable` |
| function-calls | call targets | exists: `@function`; add the suffix `.roc-special.call` if Emacs needs a call face distinct from a definition face |
| definition-names | declared names, including destructured names and `var` names | partial; add `(var_declaration name: (identifier) @variable.roc-special.definition)` and `pattern:` patterns for tuple, tag, and nominal destructuring with the same capture |
| definition-names (functions) | function definitions and annotations | exists |
| definition-names (derive) | derive candidates | exists at `highlights.scm:263-268` and `:388-398` |
| tags | tags | exists: `(tag (identifier) @constructor)` |
| modules | modules | exists: `@namespace` |
| operators | operators | exists |
| delimiters, brackets | `,` and brackets | exists |
| arrows | `->`, `=>` | exists; confirm `(arrow)` and `(fat_arrow)` both have a capture |
| lambdas | `|` in lambdas | new: `(anon_fun_expr "|" @punctuation.bracket.roc-special.lambda)`; remove the TODO conflict comment for `"|"` |
| assignments | `=` in declarations | new: `(value_declaration "=" @operator.roc-special.assignment)` |
| errors | reset `ERROR` text | new, last in the file: `(ERROR) @roc-special.error` |

## Steps

1. Read `queries/highlights.scm` and the mode rules. Confirm each "exists" row with `tree-sitter query` on `~/Projects/roc-ts-mode/tests/all_syntax_test.roc`.
2. Add each "new" and "partial" row. Place each pattern by priority: general patterns early, specific patterns later. Put `(ERROR)` last.
3. Do not delete existing captures that Helix uses.
4. Add map entries for every new capture to `query-maps/helix.json`, `neovim.json`, and `zed.json`. For `.roc-special` captures, map to the unsuffixed name if the editor has it, otherwise `""`. For `@roc-special.error`, map to `""`.
5. Run `just queries-generate` and `just ci`.

## Out of scope

- The Emacs map (plan 02).
- Changing roc-ts-mode.

## Verification

- `just ci` passes.
- For each table row, `tree-sitter query queries/highlights.scm ~/Projects/roc-ts-mode/tests/all_syntax_test.roc` shows the capture on at least one node. Record rows with no example in that file, and add a line to the test file for them in plan 03 instead.

## Done when

- [ ] Each table row has a matching base pattern.
- [ ] All three existing maps have entries for the new captures.
- [ ] `just ci` passes.
