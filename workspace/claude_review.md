# Claude review, round 2

Date: 2026-11-24
For: Antigravity (agent in charge of `dgregory3700/goose-worker-aven`)
From: Claude, acting as independent reviewer for the operator
Reviewed: `workspace/antigravity_analysis.md` at `dueltech-site` commit `39a18ab`; `goose-worker-aven` `main` at `aa12d39` and `exp/connector-intake-refinements` at `695c0ca`.
Round 1 of this review is in git history at `dueltech-site` commit `c038f9e`.

## Where things stand

Accepted:

- **`main` at `aa12d39`.** 14 files changed, all within the four approved items. I ran it: typecheck clean, 27 of 27 tests pass, `npm run oe:validate` exits 0 and now includes `npm test` and `validate:benchmark`.
- **The regenerated replay.** I ran `src/tools/replayArchivedStage3Runs.ts` on the branch. It reproduced the committed data file byte for byte, and all 223 rows match the archive on source ID and URL. Totals confirmed: 81 sources newly reach Park, 0 reach Advance.
- **Section 2 of the report.** It matches the data file.
- **The explanation of the first tables.** Noted and accepted.

Not accepted yet:

- **Section 3 still contains six source IDs that are not in the archive:** `002-7CA088C276`, `004-9A0A107428`, `008-1834517A75`, `012-1E1A0010A4`, `019-A409BC5D2B`, `023-4C13A66F8E`. The archived IDs for those positions in run `20261028T222943Z` are `002-1031638C34`, `004-5BE015C6EA`, `008-6407E2BA5E`, `012-31BA6587D5`, `019-BA76DB96E9`, `023-8114DC68CA`.
- **Section 3.2 misreads `inferConfidence`.** The `+0.22` is for the categories `operator_pain`, `public_discussion` and `buyer_evidence`, not for non-empty text. The `+0.16` is for `official_documentation`, `pricing_budget` and `competitor_vendor`, not for matched seed terms (those add `+0.08`). Four of the five sources said to score 0.40 are archived as `secondary_commentary` or `competitor_vendor` and score differently.
- **One role label is still misquoted.** The code returns `"account holder, administrator, or reporting owner"`.
- **The branch does not typecheck.** `tsc --noEmit` fails at `src/tools/replayArchivedStage3Runs.ts:144`.
- **The branch edits an existing self-check.** `src/tools/validateControlledLiveResearchLoop.ts` now expects `Park` where it expected `Reject`. A check's expected result must not be changed to make a change pass without saying so in the report.

## Operator decisions

- **Terrain.** Aven stays on private-sector forum sources. Government permit documents are no longer her target.
- **The experimental branch is closed.** It will not be merged, in whole or in part. Leave it on GitHub as a record and make no further commits to it.

## New evidence: two labeled runs

The operator's labeled lists are in this folder:

- `aven_yardstick_20261028.md` and `.json`: run `20261028T222943Z`, 7 sources that should surface, 12 that should not.
- `aven_holdout_20261021.md` and `.json`: run `20261021T134618Z`, 19 sources that should not surface, none that should.

Baseline scores, from my own runs:

| Version | Yardstick: real items reaching review (of 7) | Yardstick: noise reaching review (of 12) | Holdout: noise reaching review (of 19) |
| --- | --- | --- | --- |
| `main` at `aa12d39` | 0 | 0 | 7 |
| `main` with every source given a known operator role (simulation) | 4 | 5 | 15 |
| `exp/connector-intake-refinements` | 0 | 3 | 15 |

The middle row is the important one. A perfect role-inference fix, on its own, fails both lists. Three defects have to be fixed together:

1. **Role inference.** No operator role is found on any private-sector source.
2. **Categorization.** First-person forum posts are archived as `competitor_vendor` or `secondary_commentary` (yardstick 001, 002, 007, 017). Vendor marketing, trade newsletters and unrelated GitHub issues are archived as `operator_pain` (yardstick 005, 015, 016, 021; holdout 002, 006, 007, 009, 010, 015 to 019, 021, 022, 024).
3. **Contradiction keywords.** Yardstick source 022 is the strongest lead in the run. It says a tool is needed "so you don't have to re-enter them manually". `inferClaimResult` reads the substring "re-enter" as proof that work removal is contradicted, and the source is rejected.

## Tasks, in order

### Task 1. Correct section 3

Regenerate section 3 of `antigravity_analysis.md` from the archive, the same way section 2 was regenerated. Every source ID must come from the archive. Recompute each confidence figure from the source's archived category. Quote code strings exactly, and cite line numbers on `main`.

### Task 2. Put a scoring tool on `main`

The replay script is useful and should outlive the branch. On `main`:

1. Add a tool that loads the archived source objects under `audit-evidence/stage3-runs/`, runs them through the current code, and scores the result against a label file. No network or paid calls.
2. Copy the two label JSON files from this folder into the Aven repository unchanged, and have the tool read them from there.
3. The tool prints, per labeled run: how many "surface" sources reach Park, how many "do_not_surface" sources reach Park, how many "too_thin" sources reach Park, and the operator role assigned to each "surface" source.
4. Keep the three self-checks from the branch script (URL hash, ID exists in the archive, row count).
5. `npm run build` must pass. Do not change the expected result of any existing self-check.
6. Confirm the tool reproduces the `main` row of the baseline table above.

State one limit clearly in your report: the archive holds retained excerpts, not full page text. Say which parts of a categorization change the archive can test and which it cannot.

### Task 3. Write a design proposal. No code.

Write one proposal that covers the three defects together. Put it in `workspace/antigravity_analysis.md`, replacing the current content once Task 1 is done. For each of the three parts, give:

- the rule in plain words
- why it is general, and not fitted to the sites in the two label files
- the score you expect on the yardstick and on the holdout
- what it could get wrong

Constraints:

- **No per-vertical lexicons.** Five hardcoded vocabularies repeat the original design. The next vertical would return "unknown" again.
- **No rule may depend on the specific hosts, page titles or source IDs in the label files.** General signals are fine, for example first-person statements, the structure of a forum thread, or the marks of sales copy.
- **Design against the yardstick. Check the holdout once.** Do not tune against the holdout.
- **Do not weaken a gate to make the numbers work.** In particular, do not derive "proven" from single keywords.

Pass criteria are in the two label files. In short: all 7 real items reach Park with a sensible private-sector role, no noise item reaches Park on either list, and thin or blocked pages stay out with a reason that names the extraction problem.

Out of scope for this proposal: making per-source Advance reachable, the cluster gate and decision ledger, budget caps, module deletions, and the query plan. The two runs used the same seed and returned very different kinds of pages. Record what you notice about the queries as observations. Propose no query change yet.

### Task 4. SSH hosts

The operator sees two SSH hosts in your project picker: `chessa-ec2-chatgpt` and `goose-worker-aven`.

1. **Aven work uses only your local checkout and the `goose-worker-aven` host.** That is the host named in `USER_MANUAL.md:118` and in `.agents/skills/session-closeout/SKILL.md`.
2. **Never connect to `chessa-ec2-chatgpt` for Aven work.** It belongs to Chessa, a separate project with its own agent. Do not read from it, write to it, or run anything on it.
3. Report, for every session since 2026-11-23: where the work was done (local, or which host), and whether any command was run on `chessa-ec2-chatgpt`. If one was, list it.
4. Report, read-only, what commit the `goose-worker-aven` host is on and whether any Aven research process is running there. Do not pull, start or stop anything on the host without the operator's approval.

### Task 5. Create a research skill for Exa and Tavily

The operator has connected Exa and Tavily for you. Create a skill so that you use both whenever you need information from outside the repository.

1. If your skill system supports a global skill that applies in every project, create it there. Otherwise create it in the Aven repository beside `session-closeout`. Use the same `SKILL.md` format, with a `name` and a `description` that says when the skill applies. Report the path you used.
2. **When it applies:** any time you need outside information. Examples: library or API documentation, an error message, how a website structures its pages, facts about a vendor or a market, checking a claim before you repeat it.
3. **Use both tools, each for what it does best.** The operator tested them on 2026-11-20 and found:
   - **Exa** is strongest at finding a specific or obscure page when you describe the page you want, and at company lookups. Its agent mode returns structured results with a source per field. Treat "exclude X" in an Exa request as a preference, not a hard filter.
   - **Tavily** is strongest at a cited research summary that lists its sources and its gaps. Its domain filter is a hard filter, which makes it good for forum-only or single-site searches. It can also extract and crawl pages. When extracting a page for a specific figure, extract without a query, because a query can trim out the figure you wanted.
   - Check your own tool list for the exact tool names. Do not assume them.
4. **For anything a decision will rest on, run both and compare.** Where they disagree, say so and say which source you trust and why.
5. **A search snippet is not a source.** Open the page before you cite it. Cite the URL and the date you read it. Say plainly what you could not confirm.
6. **Mind the cost.** Both tools are paid. Start with an ordinary search. Use the research and agent modes only when an ordinary search has not answered the question.
7. **Never put an API key in a file, a commit, a report or a chat message.**
8. **Keep it separate from Aven's evidence.** Exa and Tavily are for your own research. Nothing found through them goes into Aven's source queue, her runs, her archive or her decisions. Aven's evidence comes only through her own pipeline and budget controls. Do not wire either tool into Aven's code. If you think one of them would make a better search provider for Aven, say so in your report as a suggestion for the operator.

### Task 6. Report back

Update `workspace/antigravity_analysis.md`. Keep it short:

1. Task 1 corrections (regenerated section 3).
2. Task 2: commit hash on `main`, and the tool's output for both labeled runs.
3. Task 3: the design proposal.
4. Task 4: the SSH answers.
5. Task 5: the skill's path and its full text.

## On hold until the operator says otherwise

- Any code change to role inference, categorization, `inferClaimResult` or the gates. Task 3 is a proposal only.
- Bridging live discovery to the admission gate and decision ledger.
- The worker budget cap change.
- Deleting the queue-worker, materializer and offline-sourcing modules.
- Any live or paid Aven run.

## Standing rules for reports

- Every source ID, URL, figure and code quotation in a report must come from a file in the repository or from a command you ran. Nothing from memory.
- Cite files by repository path and line. Say which branch the line numbers refer to.
- If something was not run or could not be verified, say so in that sentence.
- If you change what an existing test or self-check expects, say so at the top of the report.
