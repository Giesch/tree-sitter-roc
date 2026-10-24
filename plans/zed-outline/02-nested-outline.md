# Z02: Nested outline entries

Status: not started

## Goal

The Zed outline shows nested types and their associated items under their owners. The outline test from Z01 passes, and `npm run ci` runs it.

## Depends on

[01-outline-test.md](01-outline-test.md).

## Context

- `nominal_methods` (`grammar.js`) contains `inferred_associated_annotation` and `_module_elem` children. `opaque_type_def` and `nominal_type_def` have an optional `nominal_methods` child.
- `annotation_type_def` and `inferred_associated_annotation` expose `name: (annotation_pre_colon)`. The planned ordinary alias, opaque, and nominal type patterns require `name: (apply_type)`. Constraint aliases with `name: (where_alias_name)` are outside this plan's scope. `value_declaration` exposes `pattern: (decl_left)`.
- A signature and its definition are sibling statements: `to_str : a -> Str` and `to_str = |value| ...`. If the query captures both as `@item`, the outline shows the name 2 times.
- Zed nests entries by `@item` range containment. A query pattern does not need to state the parent.

## Resolved operator decisions (2026-10-23)

1. Emit one entry per signature/definition pair: prefer the signature and retain definitions that have no signature. Signature-only methods remain entries. Verify query-only deduplication against sibling names and positions. Entries for separate sibling declarations must have disjoint item ranges; deduplication must happen in the query, with no renderer filtering. Preserve proper containment for owner/child entries. Report a blocker if this cannot be made robust without changing scope.
2. Include `inferred_associated_annotation` (`is_eq : _`) as an outline entry.
3. Add syntax `@context` captures such as `::`, `:=`, or `:` and include them in rendered labels. Extend the test renderer and Zed mapping in Z02; required hierarchy paths continue to identify names.
4. Exclude all function-local declarations, including local types and signatures.

These decisions are recorded during Z01. Nested patterns, deduplication, context captures, and rendered context labels are Z02 implementation work.

## Steps

1. Apply the 4 resolved operator decisions above.
2. Change `queries/outline.scm`:
   - Remove the `(file ...)` anchors from the type patterns, or add patterns under `(nominal_methods ...)`. Choose the form that does not capture locals in block bodies.
   - Add patterns for signatures and value declarations under `nominal_methods`.
   - Apply decisions 1–3.
   - Update `query-maps/zed.json` for any new capture names.
3. Run `just queries-generate`.
4. Run `npm run outline:check`. The required paths pass. The snapshot check fails, because no snapshot exists.
5. Run `node tools/check-zed-outline.js --record`. Review `test/fixtures/Builtin.zed-outline.txt`:
   - Each nested type is under its owner.
   - No local name from a function body is present.
   - Signature and definition entries agree with decision 1.
   - Line ranges agree with the source for 10 sampled entries.
6. Add `npm run outline:check` to the `ci` script in `package.json`.
7. Update `queries/AGENTS.md` with the outline contract.
8. Run `npm run ci`.

## Out of scope

- Grammar changes.
- Outline queries for other editors.
- The maki indexer.

## Verification

- `npm run outline:check` passes.
- A deliberate change to `queries/outline.scm` that removes 1 nested pattern makes `npm run outline:check` fail. Restore the pattern.
- `npm run ci` passes.

## Done when

- The query, the reviewed snapshot, and the CI entry are in 1 commit.
- This plan records the 4 decisions and the snapshot review.
- [README.md](README.md) and [../README.md](../README.md) record the status.
