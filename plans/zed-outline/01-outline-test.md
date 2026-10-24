# Z01: Builtin.roc fixture and a failing outline test

Status: implemented and verified (2026-10-23); awaiting review and the Z01 commit

## Goal

A test checks the Zed outline of `Builtin.roc` and fails. The failure shows that the minimal query omits the nested items from issue #39.

## Depends on

Where-alias plans W01–W03. `Builtin.roc` parses with 0 `ERROR` and 0 `MISSING` nodes at `750e5f0`.

## Context

- `~/Projects/roc` is at `d93715878ca0258b24576a6649928df0dbea8f52` on branch `local-install`. No remote branch contains this commit. Upstream `main` is `6de96ed3afba20699688bc30ce40c062267f2382`. Record the blob hash of the copied file, because the blob hash identifies the content on any branch.
- `tools/template-queries.js` reads each `.scm` file in `queries/`. For each map in `query-maps/`, it writes `queries-generated/<editor>/queries/<family>.scm`. `assertMappingComplete` in `tools/template-queries.js` stops the generator when a map has no entry for a family. The `editorName === "emacs" && !mappingForFile` condition preserves the highlights-only Emacs exception.
- The `tools/check-expr-body-captures.js` and `tools/record-expr-body-baseline.js` scripts name their 6 query families explicitly. A new `outline.scm` does not change them.
- `tree-sitter query <query> <file>` prints 1 block for each match. Each block has a `pattern:` line and 1 `capture:` line for each capture, with `start: (row, column)` and `end: (row, column)`. The CLI prints `text:` only for single-line captures.
- Zed outline semantics:
  - Each match that has an `@item` capture makes 1 outline entry.
  - The `@name` capture gives the label.
  - An entry is a child of the nearest earlier entry whose `@item` range contains its `@item` range.
- `tools/check-operator-trees.js` has the `--self-test`, `--record`, and `--force` argument conventions. Copy them.
- The justfile has no `set dotenv-load`. `.gitignore` has no `.env` entry.

## Steps

1. Environment:
   - Add `.env` to `.gitignore`.
   - Create `.env.example` with 1 line: `ROC_COMPILER_DIR=/path/to/roc`. Commit `.env.example`. The `.env` pattern in `.gitignore` does not match `.env.example`.
   - Create `.env` with `ROC_COMPILER_DIR=/home/danknutson/Projects/roc`. Use an absolute path, because just does not expand `~` in dotenv values. Do not commit `.env`.
2. Justfile:
   - Add `set dotenv-load` at the top.
   - Add a `fixture-builtin` recipe. The recipe:
     - Stops with an error message when `ROC_COMPILER_DIR` is not set.
     - Stops when `src/build/roc/Builtin.roc` has uncommitted changes in that checkout.
     - Copies the file to `test/fixtures/Builtin.roc`.
     - Writes `test/fixtures/Builtin.roc.source` with 3 lines: `revision <HEAD>`, `branch <name>`, and `blob <git hash-object of the file>`.
   - Run the recipe. Commit `Builtin.roc` and `Builtin.roc.source`.
3. Generator:
   - In `tools/template-queries.js`, skip a family when the editor map sets the family to `false`. Keep the completeness check for all other families. Keep the Emacs rule.
   - Add `"outline": false` to `query-maps/helix.json` and `query-maps/neovim.json`.
   - Add `"outline": { "item": "item", "name": "name" }` to `query-maps/zed.json`.
   - Add a failure-path case to an existing generator self-test, or to the new test in step 5. A map with no `outline` entry must still stop the generator.
   - Update the contract in `queries/AGENTS.md`.
4. Minimal query: create `queries/outline.scm` with top-level items only. Anchor each pattern under `(file ...)`:
   - `alias_type_def`, `opaque_type_def`, and `nominal_type_def`: `@name` on the `concrete_type` of `name: (apply_type ...)`.
   - `annotation_type_def`: `@name` on the identifier in `name: (annotation_pre_colon ...)`.
   - `value_declaration` with an identifier pattern: `@name` on the identifier.
   Run `just queries-generate`. Only `queries-generated/zed/queries/outline.scm` is new.
5. Test: create `tools/check-zed-outline.js`.
   - Run the pinned CLI with `queries-generated/zed/queries/outline.scm` on `test/fixtures/Builtin.roc`.
   - Read the matches. Assert that each match has exactly 1 `@item` and exactly 1 `@name`.
   - Sort the entries by start position, and by end position in descending order. Give each entry a depth with a stack of open `@item` ranges.
   - Stop with an error when 2 entries have equal `@item` ranges. Zed's handling of this case is not verified.
   - Render 1 line for each entry: 2 spaces for each depth level, the name, and the 1-based line range of the `@item` (`Str L5-L120`).
   - Required paths: assert that the rendered outline contains these paths from issue #39:
     - `Builtin > Str > is_empty`, `Builtin > Str > concat`, `Builtin > Str > trim`
     - `Builtin > List > len`, `Builtin > List > map`
     - `Builtin > Bool > not`, `Builtin > Bool > is_eq`
     Before you commit these paths, confirm in the fixture that each path exists in the source.
   - Snapshot: compare the rendered outline to `test/fixtures/Builtin.zed-outline.txt`. If the snapshot does not exist, fail with a message that names `--record`.
   - Arguments: `--record` writes the snapshot only when it does not exist. `--record --force` replaces it. `--self-test` checks the containment and rendering code with a small inline source and query, and checks that a required path that is absent fails.
   - Add an npm script `outline:check`: `node tools/check-zed-outline.js --self-test`. Add a just recipe `outline-check`. Do not add the script to `npm run ci`.
6. Run `npm run outline:check`. The self-test passes. The required-path check fails, because the minimal query has no nested entries. Record the failure output in this plan.
7. Run `npm run ci`. It passes, because it does not run the outline test yet.

## Out of scope

- Nested outline patterns, `@context` captures, and the snapshot. Z02 does these.
- Zed extension packaging and editor rendering.

## Verification

- `just fixture-builtin` stops when `ROC_COMPILER_DIR` is not set.
- `git status` does not show `.env`. `git ls-files .env.example` shows `.env.example`.
- `just queries-generate` creates `queries-generated/zed/queries/outline.scm` and no Helix, Neovim, or Emacs outline file.
- `npm run outline:check` fails only at the required-path check, and the message names the first missing path.
- `npm run ci` passes.

## Execution record — 2026-10-23

- Implemented environment setup, fixture-copy recipe and provenance, explicit generator family opt-outs, the minimal file-child query, and the outline guard with npm/just entry points. No nested query patterns, context captures, snapshot, or CI outline entry were added.
- `.env` was ignored before creation, created with mode 600, and remains absent from `git status`. `.env.example` is not ignored and contains only the nonsecret compiler-path placeholder. The parent has staged `.env.example` for the Z01 commit; `git ls-files .env.example` now shows it. Review fixes are left unstaged for the parent to restage.
- `just fixture-builtin` copied revision `d93715878ca0258b24576a6649928df0dbea8f52`, branch `local-install`, blob `ec8982a92cf1b132c0d3af910a22e3c40237ca22`. The copied blob was checked independently. The pinned CLI parsed the copied fixture successfully (`parse --quiet`, exit 0).
- `env -u ROC_COMPILER_DIR just --no-dotenv fixture-builtin` fails with `ROC_COMPILER_DIR must be set to a Roc checkout`. A temporary Git checkout with a staged change to Builtin.roc fails with `Builtin.roc has uncommitted changes in the compiler checkout`; the real fixture remained unchanged. The initial attempted `just --unset` probe was unsupported; the corrected probe above passed.
- Confirmed required paths in the copied source: `Builtin` L1; `Str` L2222 with `is_empty` L2262, `concat` L2271, `trim` L2286; `List(_item)` L3896 with `len` L3903 and `map` L4588; `Bool` L5549 with `not` L5560 and `is_eq` L5563. Each method is in its owner's associated block, not a function-local declaration.
- `just queries-generate` passes. Only Zed gains an outline file; existing generated files remain unchanged. Generator regressions check explicit `false` opt-out success, absent-family failure, and missing-capture failure.
- CLI-backed inline tests cover multiline item captures (which omit numeric capture IDs), hierarchy, sibling exit, outer-first sorting, equal and crossing ranges, byte-column extraction, exact capture cardinality, missing required paths, and snapshot creation/replacement guards. A separate parser-valid inline fixture exercises the production generated Zed query for all five top-level declaration kinds, asserts exact item byte ranges and rendered lines, and excludes nested and function-local declarations.
- Deliberate invariants: each match has exactly one item and name; item ranges are nonempty, unique, and either contained or disjoint (never crossing); each name is nonempty, single-line, and contained by its item. Byte lines are prepared once per parse; no multiline name extraction is supported. Only the exact source path passed to the query is accepted as an output header; an unexpected `.roc` header is rejected. Z02 syntax context punctuation is also single-line.
- Review fixes add README/just task documentation and an explicit `DETACHED` provenance branch fallback. The fallback was checked in a temporary detached Git checkout; `just --list` shows both task descriptions. No duplicate stderr printing was added: the observed Node `execFileSync` error message already included captured CLI stderr. After review fixes, `npm run ci`, `node --check`, and `git diff --check` passed again; an automated subprocess assertion verified exit 1 and the exact intended missing-path stderr after all outline self-tests pass.
- `npm run outline:check` exits 1 intentionally, after its self-tests pass, with this output:

```text
> tree-sitter-roc@1.0.0 outline:check
> node tools/check-zed-outline.js --self-test

Zed outline self-tests passed (CLI matches, containment, rendering, failure paths, snapshot guards, generator opt-out/completeness)
Missing required outline path: Builtin > Str > is_empty
```

- `npm run ci` passes unchanged: 268/268 corpus parses, all 14 highlight fixtures completed, generated artifacts and queries clean, exact capture checks, 18 shared highlights, and operator full-tree/self-tests passed. Outline remains outside CI.
- Grammar, scanner, generated parser files, and operator baselines were not changed. Existing `tmp/` content was left untouched. No commit was made; the parent will review and commit Z01 before implementing Z02.

## Done when

- `.env.example`, the fixture, the source record, the generator change, the minimal query, and the failing test are in 1 commit.
- This plan records the failure output.
