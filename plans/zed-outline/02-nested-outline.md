# Z02: Nested outline entries

Status: completed and verified in the commit containing this record (2026-10-23).

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

These decisions were recorded during Z01 and are implemented in Z02.

## Implementation evidence (2026-10-23)

- Feasibility was tested with the pinned Tree-sitter 0.25.10 before production edits. An optional preceding signature plus `#not-eq?` drops unsigned definitions because an absent capture fails that predicate. Separate first-child, non-signature-predecessor, and different-name-signature patterns preserve unsigned definitions and suppress only matching pairs across line/doc comments. Explicit `_previous` captures are required: without them the pinned engine dropped consecutive definition matches in the expanded regression fixture.
- The query uses only built-in `#not-eq?`, not editor-specific predicates. Current [Zed predicate dispatch](https://github.com/zed-industries/zed/blob/main/crates/language/src/syntax_map.rs) recognizes only `has-parent?` and `not-has-parent?` as custom predicates; unknown general predicates return true. No unsupported ancestor predicate is used.
- [Zed outline capture consumption](https://github.com/zed-industries/zed/blob/main/crates/language/src/buffer.rs) creates one entry per match, takes name/context captures in source order, inserts a space for source gaps, and derives depth from containment. The test follows these rules without deduplicating. This is source-backed modeling, not a Zed UI test.
- All four decisions above are implemented. Types require `apply_type` names; constraint aliases are excluded. File/member parent patterns exclude locals. A valid local binding/signature fixture and a recovered local nominal-with-members fixture assert exclusion; the grammar does not support nominal definitions as block statements.
- `just queries-generate` is the only generation command used for query edits. Before recording, `npm run outline:check` passed self-tests and required paths and failed with `Outline snapshot missing; use --record to create it`. `--record` then created the first snapshot.
- Full snapshot audit: all 2,827 entries were independently matched to a CLI XML parse by declaration name and source range; every entry had a file/member direct parent, none had function/block ancestry, and no sibling names were duplicated. Ten direct source samples: `Encoding.FieldName.FieldNames` L68–74; `Encoding.ParseTagUnionSpec.parse` L85–94; `Str.Utf8Problem` L2226–2235; its `is_eq` L2234; `Str.is_empty` L2262; `Str.concat` L2271; `Str.trim` L2286; `List.len` L3903; `List.map` L4588; `Bool.not` L5560 (also checked `Bool.is_eq` L5563). All agree; paired definitions do not add entries.
- Deliberately removed the nested `annotation_type_def` pattern, regenerated, and observed `npm run outline:check` fail its production assertion. The direct check also failed `Builtin > Str > is_empty`. Restored the pattern, regenerated, and passed against the original snapshot without refreshing it.

## Final verification

- With Z02 staged in the actual repository, literal `npm run ci` passed (exit 0): artifact checks, parser budgets (3149 states / 905 large states / 5234140 bytes), all 268 corpus parses, all 14 highlight fixtures, generated-query checks, exact expression-body captures, 18 shared-highlight captures, operator full-tree self-tests (22 operators and 5 recovery fixtures), and the outline check. This supersedes the earlier disposable-copy verification; the generated-query clean-diff gate is resolved.
- `git diff --check` passed. Grammar, scanner, generated parser artifacts, and operator baseline are unchanged.
- Review: Kimi blind-defect review found no blocking issues. Astra pathology/test audit reported no findings and independently audited all 2,827 snapshot entries against XML/source. Fable review found no Critical/Important issues. Its concrete Minor omission of unsigned definitions after a recovered `ERROR` sibling is fixed by adding `ERROR` to both non-signature predecessor alternatives. The targeted `first = 1\n)\nunsigned_after_error = 2\nlater = 3\n` fixture intentionally produces a parse error and must retain all three names. The existing Builtin snapshot remains unchanged.
- Post-review verification: regenerated via `just queries-generate`; `npm run outline:check` and `git diff --check` pass, with the existing snapshot unchanged. After staging all review fixes, literal `npm run ci` passed again in the actual repository (exit 0), including all checks listed above; `git diff --cached --check` also passed.
- Pairing across `expect` was adjudicated against the root sibling-position contract: only adjacent signature/definition siblings pair, with line/doc comments permitted between them. Any intervening statement ends the pair. A production regression at both file and member scope expects separate `sig :` and `sig` entries when `expect` intervenes; no cross-statement name-based deduplication is performed.

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
