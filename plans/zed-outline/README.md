# Zed outline plans

These plans add a Zed `outline.scm` query that shows nested types and their associated items. The requirement comes from faldor20/tree-sitter-roc#39.

## Order

| Plan | Change | Depends on |
|---|---|---|
| [01-outline-test.md](01-outline-test.md) | Fixture, generator support for a Zed-only family, a minimal top-level query, and a failing outline test | [../where-aliases/](../where-aliases/README.md) W01–W03 |
| [02-nested-outline.md](02-nested-outline.md) | Nested outline query, reviewed snapshot, and CI entry | Z01 |

Each plan is one commit. Z01 leaves the outline test failing. Z01 does not add the test to `npm run ci`. Z02 adds it.

Status (2026-10-23): Z01 is implemented and verified, awaiting review and its separate commit. `npm run ci` passes; outline self-tests pass before the intentional missing `Builtin > Str > is_empty` failure. Z02 implementation has not started; its four operator decisions are resolved in [02-nested-outline.md](02-nested-outline.md).

## Decisions

- The fixture is `src/build/roc/Builtin.roc` from the Roc compiler. A just recipe copies it from the checkout named by `ROC_COMPILER_DIR` in a gitignored `.env` file. The test reads only the checked-in copy.
- The expected outline is a full snapshot of the outline for the fixture.
- The generator skips a query family for an editor when the editor map sets the family to `false`. The completeness check stays in effect for all other families.
- The test builds the hierarchy from `@item` range containment, with the rule that Zed uses. The test does not run Zed.

## Constraints

- Base queries in `queries/` are the source. Regenerate `queries-generated/` with `just queries-generate`. Do not edit generated files.
- The snapshot follows the baseline rules for `tools/check-operator-trees.js` in [../../AGENTS.md](../../AGENTS.md): `--record` creates the snapshot, and only `--record --force` replaces it. Review the snapshot diff before a commit. Do not refresh the snapshot to hide a regression.
- These plans do not change `grammar.js`.
