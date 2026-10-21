# 07.02: Corpus and queries without `expr_body`

Status: done (`c0f87e4bbb94555bb1cd810934c75c11f165c334`; scope follow-up `45fb54d5b4111e94e2a939fca0aa45abf749c47d`)

Completion note: Generated queries and fixtures cover direct fielded bodies. The follow-up restores declaration-body scopes across concrete expression forms. The regression guard checks query captures and geometric scope relationships, with explicit new match-branch/block scope captures; it does not test editor resolvers or rendering. Separately, the independent reviewer compared pinned-CLI HTML highlight output before and after on the sample, three highlight fixtures, and capture fixture: all five outputs were byte-identical. This does not establish editor-specific rendering or name resolution.

## Goal

Corpus and all query files match the grammar from sub-plan 01, with the same highlighting, scope, indent, and text-object coverage.

## Depends on

[01-grammar.md](01-grammar.md), uncommitted in the working tree.

## Steps

1. Run `just test-update`. Review the diff. The only change is removal of `(expr_body ...)` levels at the changed positions.
2. Update each query file. Use `grep -n 'expr_body' queries/*.scm`.
   - `highlights.scm` function-definition patterns (lines 248-256): `body: (anon_fun_expr)`.
   - `tags.scm:14`: `body: (anon_fun_expr)`.
   - `locals.scm:1`: keep `(expr_body) @local.scope` for remaining wrappers. Add scopes for nodes that now hold bodies directly: `(anon_fun_expr) @local.scope`, `(match_branch) @local.scope`, `(body_expression) @local.scope`.
   - `indents.scm:6`: add `@indent` patterns for the parents whose bodies lost the wrapper, for example `(anon_fun_expr body: (_) @indent)`. Confirm the capture semantics for each editor map before you choose the form.
   - `textobjects.scm:2`, `:16`, `:33`: use the field, for example `(anon_fun_expr body: (_) @function.inside)`.
   - `rainbows.scm:3`: replace with the nodes that now hold bodies.
3. If supertypes exist from plan 06, prefer the supertype in replacement patterns where a wildcard is too broad.
4. Run `just queries-generate` and `just ci`.
5. Commit sub-plans 01 and 02 together.

## Out of scope

- roc-ts-mode. Its rules adapt in `emacs-mode/02` and `emacs-mode/03`.

## Verification

- `just ci` passes.
- `tree-sitter highlight` on `test/highlight/*.roc` and one sample from `samples/` produces the same output before and after. Save the "before" output from the plan 06 commit.

## Done when

- [x] No query references a removed wrapper position.
- [x] Independent pinned-CLI HTML highlight comparison is byte-identical on all five checked files; the committed guard separately checks exact captures.
- [x] `just ci` passes.
