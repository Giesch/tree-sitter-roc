# Roc editor queries

Last verified: 2026-10-19

## Purpose and boundaries
Base queries preserve editor captures as the syntax tree evolves. Edit these sources and `query-maps/<editor>.json`, then regenerate `queries-generated/`; generated editor copies are not independent sources.

## Contracts
- `tools/template-queries.js` maps captures for Helix, Neovim, and Zed. Each editor map covers every base capture; null, empty-string, or false mappings remove a capture. Generated non-comment lines have trailing whitespace trimmed.
- `highlights.scm` captures `try_operator` and `"return"` as `@keyword.control.return`; the Neovim mapping yields `@keyword.return`. `tools/check-neovim-try-highlight.js` checks actual postfix and infix `?` captures using the local CLI, not just query text.
- Tag-type names use `tag_type name: (tag_name)` with `@type.enum.variant`. Keyword queries match anonymous tokens. Annotation/definition queries use fields and sibling statements rather than the removed annotation nesting.
- Direct expression bodies retain their prior capture ranges even without `expr_body` nodes. Keep retained wrapper captures for contexts that still expose them.
- `locals.scm` captures lambdas, match branches, and blocks as scopes. Declaration-body alternatives enumerate `_expr_inner` subtypes except `anon_fun_expr` and `body_expression`, which already have scope captures. Keep this list synchronized when expression forms change; the regression guard checks it against generated node types.
- Match-branch scopes isolate sibling branches. Current pattern identifiers remain references, not local definitions; these queries do not promise full name resolution.

## Verification
- `npm run queries:check` regenerates output, checks Neovim try captures (including failure-path self-tests), and requires no generated diff.
- `node tools/check-expr-body-captures.js --self-test` checks all six base query families against `tools/expr-body-captures-expanded-baseline.json`: capture names, byte-column ranges, multiplicity, nested scopes, and branch isolation. Intentional contract changes require reviewing captures, not blindly rerecording the baseline.
- These checks establish CLI query behavior, not editor rendering. Verify editor-specific behavior separately when making such claims.
