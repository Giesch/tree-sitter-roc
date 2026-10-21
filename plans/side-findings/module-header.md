# S01: Module header with a bracket list

Status: done (b1375d35e851e0b40c1daf1b15e02c0d06d0758c)

## Goal

`module [a, B]` parses without `ERROR`, and a corpus test covers it.

## Depends on

None.

## Context

- `module_header: seq("module", $.exposes_list)` (`grammar.js:517`).
- `exposes_list: seq("exposing", seq("[", sep_tail($.ident, ","), "]"))` (`grammar.js:535`). It requires the keyword `exposing`. Imports use the same rule (`grammar.js:551`), where `exposing` is correct.
- `exposed_list: seq("{", sep_tail($.ident, ","), "}")` (`grammar.js:527`) uses braces. It can be unused.
- Observed: `module [f]` as the first line gives `(ERROR [0, 0] - [0, 6])` and an `expr_body` with a `list_expr` for `[f]`.
- No corpus test contains a `module [...]` header. `grep -n '^module ' test/corpus/*.txt` returns nothing.

## Steps

1. Confirm the current Roc module header syntax. Check Roc sources or docs: the Roc repository's `crates/compiler/parse` header parser, or the language reference. Record the source in the commit message. Expected form: `module [name, Type, ...]`, with an optional trailing comma.
2. Check whether the confirmed syntax allows anything more in a module header (for example `exposing` as an older form). Support only the confirmed current forms.
3. Add a rule for the bracket list, for example `module_exposes: seq("[", sep_tail($.ident, ","), "]")`, and use it in `module_header`. Leave `exposes_list` unchanged for imports.
4. Check `exposed_list` with `grep -n 'exposed_list' grammar.js queries/*.scm`. If only its definition matches, delete it.
5. Run `tree-sitter generate` and `just parser-metrics-check`.
6. Add tests to `test/corpus/module.txt`:
   - `module_header_empty`: `module []`.
   - `module_header_values_and_types`: `module [a, B]` then one declaration.
   - `module_header_trailing_comma_multiline`: entries on separate lines with a trailing comma.
7. Run `just test`. Existing tests do not change. Run `just test-update` only for the new tests, then check their trees by hand.
8. Grep `queries/*.scm` for `module_header` and `exposes_list`. Add `(module_header (module_exposes (ident) @special.roc-special.exposed))` to `highlights.scm` if exposed identifiers in imports have that capture. Add map entries if a new capture name appears.
9. Run `just ci`.

## Out of scope

- `app`, `package`, and `platform` headers.

## Verification

- `just ci` passes.
- `tree-sitter parse` on `module [a, B]\n\na = 1\n` has no `ERROR`.

## Done when

- [x] Header syntax source recorded in the commit message.
- [x] Three corpus tests pass.
- [x] `just ci` passes.
