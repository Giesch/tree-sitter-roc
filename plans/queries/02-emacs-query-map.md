# Q02: Emacs query map and generator support

## Goal

`just queries-generate` writes `queries-generated/emacs/queries/highlights.scm`. The file uses Emacs face names as captures, and it marks each run of patterns with an Emacs font-lock feature. roc-ts-mode can load it directly.

## Depends on

[01-shared-query-parity.md](01-shared-query-parity.md).

## Context

- `tools/template-queries.js` maps captures line by line. Each map file is `{ "<query file>": { "<capture>": "<editor capture>" } }`. Values `""`, `null`, and `false` remove the capture.
- Emacs `treesit-font-lock-rules` needs a `:feature` for each query. Users toggle features with `treesit-font-lock-level` and `treesit-font-lock-recompute-features`. The mode's feature names and levels are in `roc-ts-mode.el:617-621`.
- In Emacs, a capture name is a face or a function. With `:override t`, a later rule overrides an earlier one. The base file has the same order semantics.
- Emacs faces used by the mode: `font-lock-*-face`, `roc-ts-crash-face`, `roc-ts-question-mark-face`, `default` (for `ERROR`).
- Emacs in this environment is 30.1. Base queries use `#match?`, `#not-match?`, and `#eq?`. Emacs support for these in string queries is not confirmed.

## Steps

1. Check predicate support. In a disposable Emacs daemon (use the `testing-emacs-daemons` skill) with the roc grammar installed, evaluate:
   ```elisp
   (treesit-query-compile 'roc "((identifier) @x (#match? @x \"^a\"))" t)
   (treesit-query-compile 'roc "((identifier) @x (#eq? @x \"a\"))" t)
   (treesit-query-compile 'roc "((identifier) @x (#not-match? @x \"^a\"))" t)
   ```
   Record which forms compile. For each one that fails, find the Emacs form (`#match`, `#equal`, `#pred`) in the Emacs manual node "Pattern Matching".
2. Extend the map format, only for maps that need it. Accept an object value:
   ```json
   "keyword.control": { "capture": "font-lock-keyword-face", "feature": "keywords" }
   ```
   A string value keeps today's meaning. Other maps do not change.
3. Add an optional top-level map key `"$emacs"` or a generic `"$options"` with:
   - `"featureSections": true`: group output into feature sections.
   - `"predicates": { "#match?": "#match", ... }`: rewrite predicate names, from step 1.
4. Implement feature sections in `tools/template-queries.js`:
   - Split the query file into top-level patterns. Track parentheses and brackets, and skip text in strings and `;` comments.
   - Find each pattern's feature from its captures. If one pattern has captures with different features, fail with the pattern text in the error message.
   - Keep the original pattern order. Start a new section each time the feature changes. Write `;; feature: <name>` before each section.
   - Patterns whose captures are all removed are dropped.
5. Add `query-maps/emacs.json`. Cover `highlights` only. Other query files: add a map section only if the generator requires one for every file; map all their captures to `""` and skip their output if the generator supports that, or write them unchanged.
6. Fill the map from the mode's current rules (`roc-ts-mode.el:207-417`). Use the same face and feature as the mode for each capture. Required special cases:
   - `keyword.control.exception` → `roc-ts-crash-face`, feature `keywords`.
   - `keyword.control.return.roc-special.try` → `roc-ts-question-mark-face`, feature `early-return`.
   - `roc-special.error` → `default`, feature `errors`.
7. Run `just queries-generate`. Run `just queries-check` after a commit.
8. Load the generated file in the disposable daemon. Compile each section with `treesit-query-compile`. Every section must compile.

## Out of scope

- Changing roc-ts-mode to load the file (`emacs-mode/02`).
- Indent, locals, and text-object queries for Emacs.

## Verification

- `just ci` passes.
- `grep -c '^;; feature:' queries-generated/emacs/queries/highlights.scm` is at least the number of features in the mode's feature list.
- Every section compiles in Emacs 30.1 (step 8).
- `git diff queries-generated/helix queries-generated/neovim queries-generated/zed` is empty after this plan. The generator change does not affect other editors.

## Done when

- [ ] Generator supports object map values and feature sections.
- [ ] `query-maps/emacs.json` exists.
- [ ] Generated Emacs file compiles section by section.
- [ ] Other editors' output is unchanged.
