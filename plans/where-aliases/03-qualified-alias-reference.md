# W03: Qualified where-alias references

Status: done (W03 commit containing this record)

## Goal and scope

Accept lowercase receivers followed by one or more uppercase alias segments, such as `a.Holder.Show` and `a.Str.Utf8Problem`, with optional contiguous general type arguments. Depends on W02, baseline `e0c78286d1b1730d7ce5ac8df8252f7d53da1c3d`.

Qualified headers, header argument validation/spacing, name resolution, and semantic alias validation remain out of scope. In particular, preserve W01's documented header divergences: upstream headers have exactly one uppercase segment and accept spaced parentheses with variable-only arguments.

## Latest-main gate and evidence

Astra fetched `origin/main` in the Roc repository at **2026-10-23T22:10:20Z**, exact revision **`04d6133c766813aae75496d1a4d089506cd59ac2`**. Implementation read that exact ref with `git show`, not working-tree files.

Source-derived evidence at that revision:

- `src/parse/Parser.zig:3332–3349`: `readWhereAliasQualificationChain` loops over `DotUpperIdent` and `NoSpaceDotUpperIdent`.
- `Parser.zig:5941–5996`: lowercase receiver; the lowercase dotted method branch requires a colon. Line 5971 attaches general type arguments only for `NoSpaceOpenRound`.
- `src/parse/tokenize.zig:779–808`: space, tab, LF, CRLF and `#` comments are trivia; bare CR has a diagnostic. Lines 1328–1339 track preceding whitespace. Lines 1341–1428 inspect the byte immediately after a dot; trivia before each dot is allowed, trivia after each dot is not. Lines 1623–1625 distinguish spaced `(`.
- `Parser.zig:6154–6175` and `1113–1157` establish the out-of-scope header behavior above.

Committed snapshot evidence (same exact revision): `test/snapshots/where_alias/where_alias_qualified_local_type.md` and `where_alias_qualified_spaced_segments.md` have NIL problems. `where_alias_qualified_builtin_nested_type.md` has a syntax-valid PARSE with a semantic **Not a Where Alias** report, not a syntax failure. All three exact SOURCE sections are corpus inputs.

Astra could not execute compiler probes under its scratch policy. The upstream conclusions are source-derived plus committed snapshot evidence, **not a fresh compiler execution**. Fresh executions below exercise this Tree-sitter parser only.

## Accepted/rejected matrix

All entries refer to references inside `where [...]`, not headers.

| Form | Syntax | Evidence / corpus |
| --- | --- | --- |
| `a.Holder.Show`, `a.Str.Utf8Problem` | Accept | qualification loop; exact snapshot SOURCE cases |
| `a.One.Two.Three([])` | Accept | repeated uppercase segments; general arguments |
| `a.Holder.Show(err, [Bad, ..errs],)` | Accept | contiguous arguments, optional trailing comma |
| `a .Holder .Show` | Accept | spaced token variants; exact spaced snapshot |
| space, tab, LF, CRLF, or `#comment` plus LF before first/later dot | Accept | tokenizer trivia; explicit corpus coverage at both dots |
| space, tab, LF, CRLF, or `#comment` plus LF after first/later dot | Reject | dot must touch uppercase identifier; ten negative cases |
| `A.Holder.Show` | Reject | receiver must be lowercase |
| `a.holder.Show`, `a.Holder.show` | Reject | alias segments must be uppercase; lowercase method needs colon |
| `a Holder.Show`, `a.Holder Show` | Reject | missing first/later dot |
| `a.Holder.Show (err)` | Reject | spaced `(` does not attach reference arguments |
| `a.show : a -> Str` mixed with qualified references, either order | Accept | method branch remains valid |

Deferred pre-existing discrepancy: this grammar's global extras accept bare CR, while the upstream tokenizer diagnoses it. W03 does not change that behavior; aligning it requires separate work and is outside this constraint-only change.

## Implementation and tree contract

`where_alias_ref` now repeats the existing dot plus immediate uppercase token. Each uppercase segment is a separate `concrete_type` leaf after the existing `bound_variable`; optional `type_args: (apply_type_arg ...)` is unchanged from W02. Generic `concrete_type` is deliberately not reused: its non-immediate uppercase segments permit after-dot extras. Existing type captures therefore remain applicable without query changes. No scanner or conflict declarations changed. `src/node-types.json` is unchanged because the existing repeated-child contract already accommodates these leaves.

New tests live in `test/corpus/constraint_alias_qualified_references.txt` to isolate the targeted update and preserve every pre-existing corpus byte. Valid expectations were generated with the pinned CLI and fields shown. Sixteen negative recovery expectations were inspected and entered explicitly.

## Scoped budget exception

The initial guard failed against W02's 3146 states / 905 large states / 5232408 bytes. The operator explicitly approved exact measured ceiling bumps for W01–W03 within a cumulative 2% of the ORIGINAL **3144 / 905 / 5231479**, not 2% per fix. This is the scoped exception to the generic stop rule in [../grammar/README.md](../grammar/README.md); it does not authorize other growth.

Measured W03: **3149 states / 905 large states / 5234140 bytes**. Change from W02: **+3 / +0 / +1732**. Cumulative from original: **+5 / +0 / +2661** (about 0.1590% / 0% / 0.0509%). Only exceeded ceilings were raised, exactly to measurements; no speculative headroom. Generation reports no new conflicts. Stop if any metric exceeds its original ceiling times 1.02.

## Verification — 2026-10-23

- Pinned CLI generation succeeds; metrics guard passes with exact measured ceilings.
- Guarded `npm run ts:test`: **268/268 corpus parses**, all 14 highlight fixtures completed. Twenty-one new cases cover the matrix; previous cases remain unchanged.
- All three latest snapshot SOURCE sections parse cleanly. Semantic rejection of the builtin nested type remains syntax success.
- Historical Builtin at `6de96ed3afba20699688bc30ce40c062267f2382` and latest Builtin at `04d6133c766813aae75496d1a4d089506cd59ac2` were extracted with `git show`: both parse exit 0, **0 ERROR / 0 MISSING**.
- Queries regenerate unchanged; query generator self-tests, node-type self-tests, exact expression-body captures (189 / 69 / 41 / 139 / 512 / 16), 18 shared highlights, and operator full-tree/self-tests pass.
- Final literal `npm run ci` passes with exit 0 after explicit staging, including generated-artifact and query clean-diff gates, 268/268 corpus parses, all 14 highlight fixtures (13 assertions in the reference fixture), exact captures, shared parity, and operator regressions/self-tests. The initial unstaged generated-diff gate failure is resolved.
- SHA-256 checks before/after CI regeneration confirm all three generated artifacts are byte-stable. `git diff --check` passes. All three snapshot SOURCE sections were compared byte-for-byte with `git show`; the new corpus retains four CRLF byte pairs. Full hand-written/generated-grammar diff and all corpus trees were inspected; generated parser churn reflects the new repetition symbol and state/table renumbering.

## Final review and completion

All requested final reviews completed, including the successful Fable retry, with no unresolved Critical or Important findings. The parent accepted separate `concrete_type` leaves as compatible with the plan and existing schema, and accepted the isolated corpus file. Followups add precise type assertions on both qualified uppercase leaves plus their argument (13 reference-fixture assertions), expand all sixteen negative trees without changing recovery or fields, and explain constraint-dot adjacency in a grammar comment. No further grammar semantics, scanner, query, or operator baseline changes were made.

The operator authorized a permanent `.gitattributes` exception only for `test/corpus/constraint_alias_qualified_references.txt`: `-text` preserves its intentional mixed line endings while the repository-wide LF policy remains unchanged. The first ordinary restage retained the normalized cached blob; explicit path-only `git add --renormalize` corrected it. Working and staged corpus bytes are identical and both contain exactly four CRLF pairs; cached attributes report `text: unset`. Final CI was rerun successfully after this correction. Commit scope is the eleven intended implementation/test/documentation paths plus `.gitattributes`; unrelated `tmp/` and the bell-prefixed `:w` file remain untracked and preserved.
