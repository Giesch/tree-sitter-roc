# S05: Verify import-exposure highlight precedence

Status: planned

## Goal

Plain lowercase and uppercase names in import exposure lists receive the intended exposed-name highlight capture, rather than generic variable or namespace captures.

## Depends on

- S01, `module-header.md`, provides the demonstrated module-export regression and its highlight fixture.
- No dependency on S04, `module-exposure-forms.md`. Prefer this task before S04; S04 then extends these tests to richer item forms. If S04 already supplied the plain-import tests and fix, verify that work instead of duplicating it.

## Context

- `queries/highlights.scm` currently captures the enclosing `ident` in the import pattern: `(import_expr (exposing ((ident) @special.roc-special.exposed)))`. Confirm the current pattern before editing.
- S01's module-export test demonstrated that an enclosing-node capture can lose to a generic capture on its `identifier` child. Capturing the actual `identifier` or `module` child fixed that regression.
- The corresponding import failure is suspected, not yet demonstrated. Establish actual behavior with a regression test before changing the query.
- `test/highlight/module_exposes.roc` is the existing example for exposed-name assertions. Base queries are authoritative; editor mappings generate `queries-generated/`.

## Steps

1. Read the current import grammar, highlight query order, editor maps, and highlight-test conventions. Construct a valid import with both a lowercase exposed name and an uppercase exposed name. Include a declaration if the file grammar requires it.
2. Add highlight assertions for both names using `special.roc-special.exposed`. Inspect the parse tree to confirm the fixture actually exercises import exposure nodes without errors. Run the test against the unchanged query and record the result.
3. If the test reproduces the suspected precedence failure, change only the import-exposure pattern to capture the relevant identifier children, following the demonstrated module-export approach. Preserve query priority and ordinary import-path highlighting. If the test already passes, retain the regression coverage and record that the suspected defect was not reproduced; do not make a speculative query change.
4. Assert that the surrounding import path retains its existing highlighting and that exposure punctuation is not included in the exposed-name capture. Keep module-level aliases unchanged.
5. Regenerate the editor queries. Check each editor's existing mapping: a deliberately removed capture is not a requirement to introduce a new editor style.
6. Run the verification below, commit the focused change, and record whether the original query failed. Cross-reference this result from S04 so plain-import coverage is not duplicated.

## Out of scope

- New grammar forms, wildcard exports, item aliases, or qualified exposure items; these belong to S04.
- General import-path highlighting redesign or changes to editor capture-map policy.
- Emacs font-lock rules, grammar pins, publishing, or compatibility code.

## Verification

- Lowercase and uppercase plain exposed names have explicit passing highlight assertions.
- The fixture parses without unexpected `ERROR` or missing nodes.
- If a query fix is necessary, the new test fails with the original pattern and passes with the corrected pattern.
- Existing module-export assertions and import-path highlighting continue to pass.
- Generated queries match the base query and editor maps.
- `just ci` passes with the repository-pinned CLI first on PATH. Use intended staged outputs or a disposable index for generation-diff checks; preserve unrelated work.

## Done when

- [ ] The suspected failure is reproduced or explicitly ruled out by a regression test.
- [ ] Both exposed-name cases pass without changing unrelated highlighting.
- [ ] Generated queries and `just ci` pass.
- [ ] The implementation commit records the observed behavior and focused fix, if needed.
