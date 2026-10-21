# 02: Named tag name in `tag_type`

Status: done (`85f333ef4ea2caa8fa77ef6aa8856b1c286f0c3c`)

Completion note: Corpus, named `tag_name` field queries, highlight fixture, and CI checks are covered by the commit.

## Goal

`tag_type` exposes its constructor name as a named node, `tag_name`, under the `name:` field. Queries can then highlight the name without the payload types.

## Depends on

[01-try-operator.md](01-try-operator.md) (ordering only).

## Context

- Rule: `tag_type: seq(field("name", $._upper_identifier), optional($._apply_type_args))` (`grammar.js:723`).
- `_upper_identifier` is a hidden token. The query `(tag_type name: _)` fails with "Impossible pattern".
- `queries/highlights.scm:7` and `:66` capture the whole `(tag_type)`. For `Ok(I64)`, that includes `(I64)`.
- roc-ts-mode computes the name range by hand in `roc-ts--fontify-tag-type`. The mode adapts later in `emacs-mode/02`.
- Example tree for `T : [Ok(I64), Err]`:
  `(tags_type (tag_type type_args: (apply_type_arg ...)) (tag_type))`.

## Steps

1. Change the rule to `seq(field("name", alias($._upper_identifier, $.tag_name)), optional($._apply_type_args))`.
2. Run `tree-sitter generate` and `just parser-metrics-check`.
3. Run `just test-update`. Expected diff: each `(tag_type` gains a first child `name: (tag_name)`. No other changes.
4. Update `queries/highlights.scm`:
   - Remove `(tag_type)` from the `@type` list at line 7.
   - Replace `(tag_type) @type.enum.variant` with `(tag_type name: (tag_name) @type.enum.variant)`.
5. Grep the other query files for `tag_type` and update them.
6. Add a highlight assertion to a new file `test/highlight/tag_types.roc`:
   ```roc
   T : [Ok(I64), Err]
   #    ^^ type.enum.variant
   #       ^^^ type.builtin
   ```
   Check the actual capture for `I64` with `tree-sitter highlight` and use it.
7. Run `just queries-generate` and `just ci`.

## Out of scope

- Renaming `tag_type`.
- Tag names in expressions and patterns. Those use `(tag (identifier))` already.

## Verification

- `just ci` passes.
- `tree-sitter query` with `(tag_type name: (tag_name) @n)` on `T : [Ok(I64), Err]` returns `Ok` and `Err` only.

## Done when

- [x] `(tag_type name: (tag_name))` appears in the corpus.
- [x] Highlight test passes.
- [x] `just ci` passes.
