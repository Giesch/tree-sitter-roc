# 04.02: Corpus and queries for sibling annotations

Status: done (`35b9fc3d417d872f3158994e2c544dfc2b18a86c`)

Completion note: Block annotation fixtures include function and `where` forms plus a separate annotation before a mutable declaration. Inline `var count : I64` is not supported. Both implementation reviewers verified that existing corpus changes only move annotations to sibling positions; new fixtures additionally pin record parsing. Independently probed comma-less record-like inputs can now parse as annotation blocks, but this did not change existing corpus trees beyond the intended sibling moves.

## Goal

The corpus and queries reflect sibling annotations in blocks.

## Depends on

[01-block-statements.md](01-block-statements.md), uncommitted in the working tree.

## Context

- Corpus files that contain `annotation_type_def`: `test/corpus/module.txt`, `derived_methods.txt`, `value.txt`, `nums.txt`, `tag_destructuring.txt`, `types.txt`, `examples.txt`, `modern_syntax.txt`.
- Only block-level trees change. Module-level trees already use siblings.
- `queries/highlights.scm:388-398` matches an inferred annotation followed by a `value_declaration` sibling inside `nominal_methods`. It does not change.

## Steps

1. Run `just test-update`.
2. Review the diff. In each changed tree, the only change is: `(value_declaration (annotation_type_def ...) (decl_left ...) ...)` becomes `(annotation_type_def ...) (value_declaration (decl_left ...) ...)`. The same applies to `var_declaration`. Any other change is a regression; go back to sub-plan 01.
3. Add tests to `test/corpus/block_statements.txt`:
   - `block_annotation_before_declaration`: the example from sub-plan 01.
   - `block_annotation_with_function_type`: `g : I64, I64 -> I64` followed by `g = |a, b| a + b` inside a block.
   - `block_annotation_with_where`: an annotation with `where [a.to_str : a -> Str]` followed by its declaration.
   - `block_var_annotation`: `var count : I64` form only if the language allows it; check `test/corpus` for existing `var` annotation examples first.
4. Grep `queries/*.scm` for patterns that expect an annotation inside `value_declaration`. Fix each one to use siblings.
5. Run `just queries-generate` and `just ci`.
6. Commit sub-plans 01 and 02 together.

## Out of scope

- roc-ts-mode changes. The indent rule at `roc-ts-mode.el:467` adapts in `emacs-mode/03`.

## Verification

- `just ci` passes.
- `grep -c 'annotation_type_def' test/corpus/*.txt` totals are unchanged or higher. A drop means a lost test.

## Done when

- [x] Existing corpus trees change only by annotation sibling moves; added regression fixtures cover the required forms and record controls.
- [x] Block tests cover the listed forms; inline `var count : I64` is not valid syntax (see completion note).
- [x] `just ci` passes.
