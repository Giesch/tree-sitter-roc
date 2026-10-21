# S04: Additional module-export and import-exposure forms

Status: planned

## Goal

Parse the wildcard, renamed, and qualified items supported by current Roc in module export lists and import exposure lists. Highlight the names in these items without losing the exposed-name capture to generic identifier captures.

Module headers export names; import exposure lists bring names into scope. Verify both contexts rather than assuming that every item form is legal in both.

## Depends on

- S01, `module-header.md`, implemented in `b1375d35e851e0b40c1daf1b15e02c0d06d0758c`.
- No dependency on S02 or S03. This is follow-up work, not a prerequisite for the existing numbered grammar plans. Run it separately from other grammar edits because parser generation touches shared files.
- Prefer [import-exposure-highlights.md](import-exposure-highlights.md) first. Reuse its plain-import regression tests and any capture fix, then extend coverage to richer forms; do not duplicate that investigation if it is already complete.

## Context

- `grammar.js` currently defines `module_exposes` as a bracket list of `ident` and `exposes_list` as `exposing` followed by a bracket list of `ident`. `ident` accepts only a plain lowercase identifier or uppercase module name.
- `_import_body` aliases `exposes_list` to `exposing` and separately supports an alias for the imported module. An alias on an exposed item is distinct from that module-level alias.
- S01 review found that Roc's `parseExposedItemTokens` accepts additional item forms such as `Foo.*`, `x as y`, and `Foo.Bar`. The same parser is called from `parseExposedCollectionTokens`. These are candidates to verify in each context, not a complete specification of legal combinations.
- Versioned starting source: https://github.com/roc-lang/roc/blob/601b54c07c3d9def624198b719a14ba176c6ef6e/src/parse/Parser.zig . Inspect `parseModuleHeaderTokens`, the import parser, `parseExposedCollectionTokens`, and `parseExposedItemTokens`, plus their tests. Record the exact revision used for implementation.
- `queries/highlights.scm` captures `identifier` and `module` children under module-export `ident` nodes. S01's regression test showed that capturing the enclosing `ident` instead can lose to generic leaf captures. The existing import-exposure query still captures the parent; a corresponding import failure is suspected but has not been demonstrated.
- Base queries are authoritative. Regenerate `queries-generated/` with `just queries-generate`; do not edit generated queries manually.
- Use the repository-pinned Tree-sitter CLI (`node_modules/.bin/tree-sitter`, currently 0.25.10). A different executable on PATH previously generated malformed node metadata. Put the pinned CLI first on PATH for `just` commands.

## Steps

1. Confirm the supported syntax from versioned Roc parser source and fixtures. Build a small matrix for module exports and import exposures covering plain names, qualified names, wildcards, and item aliases. Check permitted alias targets, qualification depth, whitespace, and whether wildcard-plus-alias combinations are legal. Record supporting source links and separate syntax acceptance from name-resolution requirements.
2. Add corpus cases for each confirmed form in each applicable context. Include mixtures with existing plain names, multiline lists, comments, and trailing commas. Add representative invalid combinations based on upstream rejection behavior. Include a declaration after module headers, as required by this grammar's existing file rule.
3. Extend the exposure-item grammar narrowly. Reuse an item rule across the two contexts only where the confirmed syntax agrees. Preserve existing plain-name parsing and import-module aliases; do not repurpose the module-level alias rule for item aliases. Keep changes to unrelated headers and general expression syntax out of this patch.
4. Inspect the resulting node shapes and affected queries. Add explicit source-name, path, and alias distinctions where needed for consumers; document the chosen shape in corpus expectations. Check conflicts and parser metrics rather than raising the parser budget automatically.
5. Extend highlight coverage to the new item forms. Assert exposed-name captures on the intended identifier spans, including alias names, and appropriate existing captures for `as`, dots, and wildcard punctuation. Add plain-name import assertions first to determine whether the suspected parent-capture issue is real; fix it if demonstrated. Keep captures on the relevant leaves so generic captures cannot silently override them. Reuse existing capture names and editor mappings where possible.
6. Regenerate parser and editor queries with the pinned tooling. Review corpus expectations and generated metadata; verify `src/node-types.json` is a top-level array. Existing corpus expectations should change only where the new exposure-item structure requires it.
7. Run the verification below. Record supported forms, deliberate exclusions, and source citations in the implementation commit. Note any editor-query or indentation consequences for the later grammar-pin update; do not change the mode or its pin in this task.

## Out of scope

- `app`, `package`, and `platform` headers.
- New import path, package resolution, or module-level alias syntax.
- Semantic validation that exported names exist or that imported names resolve.
- Compatibility with historical Roc syntax not confirmed by the chosen source revision.
- roc-ts-mode changes, grammar tagging, publishing, or pin updates.
- General query cleanup unrelated to module exports and import exposures.

## Verification

- Every supported form has an explicit corpus test in each applicable context; representative invalid forms retain an error rather than silently parsing as a valid exposure list.
- Existing plain-name, empty-list, multiline, and trailing-comma module-header tests pass. Existing import-module aliases continue to parse.
- Highlight assertions cover plain names, qualified items, wildcard items, and aliases where supported, including lowercase and uppercase names. The plain import test establishes whether the previously suspected capture-precedence issue exists.
- `just parser-metrics-check` and `just ci` pass with the pinned CLI first on PATH. For generation-diff checks, compare against the intended staged outputs or a disposable index, not unrelated unstaged work.
- Generated parser metadata is a top-level array, and generated queries match the base queries and maps.
- Inspect representative parse trees for unexpected `ERROR` or missing nodes; passing highlight assertions alone does not establish parse correctness.

## Done when

- [ ] A source-backed syntax matrix identifies the supported forms in both contexts.
- [ ] All confirmed in-scope forms parse and have corpus coverage.
- [ ] Exposed names and aliases have passing highlight regression tests.
- [ ] Existing plain-name and module-alias behavior remains covered and passing.
- [ ] Parser metrics and `just ci` pass with the pinned tooling.
- [ ] The implementation commit records syntax sources and any downstream editor follow-up.
