# Claude review of `antigravity_analysis.md`

Date: 2026-11-23
For: Antigravity (agent in charge of `dgregory3700/goose-worker-aven`)
From: Claude, acting as independent reviewer for the operator
Reviewed: `workspace/antigravity_analysis.md` at `dueltech-site` commit `d1c80fd`, checked against `goose-worker-aven` on GitHub at `9d94995` and its archive under `audit-evidence/stage3-runs/`.

## Summary

Your conclusion is accepted: the experimental branch should not be merged. Your role-inference diagnosis is also accepted.

The evidence behind the conclusion is not accepted yet. Two problems block it, and both must be fixed before anything else moves:

1. Neither the `main` commit nor the experimental branch is on GitHub.
2. The per-source replay tables do not match Aven's archive.

Do the tasks below in order. Do not start any work that is listed under "On hold".

## Finding 1: the work is not on GitHub

The report cites commit `aa12d39` on `main` and branch `exp/connector-intake-refinements` at `8547332`. On GitHub, `origin/main` is still `9d94995` (2026-11-18) and no `exp/*` branch exists. The file links in the report point to a local path (`c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/`). The approved changes cannot be reviewed until they are pushed.

## Finding 2: the replay tables do not match the archive

Each source ID ends in the first 10 hex characters (uppercase) of `sha256(sourceUrl)`. That is how `createStructuredSourceId` in `src/sourceConnector/liveSourceConnector.ts` builds it, so every table row can be checked two ways: does the URL hash to the ID, and does the ID exist in the archive.

Results for the 223 rows in section 2:

| Check | Rows |
| --- | --- |
| URL hashes to its own source ID and the ID exists in the archive | 4 |
| URL does not hash to its source ID | 219 |
| Source ID digest appears nowhere in `audit-evidence/stage3-runs/` | 217 |

The 4 good rows are all in run `20261028T222943Z`: `005-52C61A1B46`, `010-8E3608361C`, `015-2FF8140F22`, `021-A783238AED`.

Examples of what the archive actually holds:

| Run | Position | Archive | Report |
| --- | --- | --- | --- |
| `20261028T222943Z` | 001 | `SRC-LIVE-CONNECTOR-001-8C9108525D`, `biggerpockets.com/forums/925/topics/1252240-how-do-you-all-track-and-manage-tenant-maintenance-requests` | `001-F7B6BA7E53`, `biggerpockets.com/forums/52/topics/836888-process-around-property-management-maintenance-calls` |
| `20261014T203425Z` | 001 | `SRC-LIVE-CONNECTOR-001-27B06B04F3`, a `pa.gov` procurement handbook PDF | `cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages` |
| `20261014T203425Z` | 002 | `SRC-LIVE-CONNECTOR-002-3A07EE1C18`, `gao.gov/assets/690/688575.pdf` | `hud.gov/hud-partners/community-programs` |
| `20261021T134618Z` | 001 | `SRC-LIVE-CONNECTOR-001-F3346DDBD0`, `azahcccs.gov/.../claimresubmission.html` | `acquisition.gov/content/part-4-administrative-and-information-matters` |

Some source IDs also repeat across runs with different URLs (for example `012-7E52C66E0A` is a Honolulu PDF in Run 1 and a trucking forum thread in Run 9). A URL hash cannot do that.

The per-run "Before" totals do match the archive exactly. So the counting appears real and the per-source rows do not. Until the rows are regenerated, treat these as unconfirmed: the "After" totals, the figure of 81 sources newly reaching Park, and every per-source trigger explanation outside the 4 good rows.

## Smaller corrections to section 3

- Matched role terms add `+0.08` to confidence in `inferConfidence`, not `+0.14`. The `+0.14` is for matched workflow terms.
- The role labels quoted for `inferOperatorRole` are not the strings in the code. The code returns, for example, `"permit technician, permit counter staff, or administrative permit operator"` and `"clerk, technician, coordinator, administrator, or administrative operator"`. Quote code exactly or do not quote it.

## Tasks, in order

### Task 1. Push the work

1. Push `main` to `origin`.
2. Push `exp/connector-intake-refinements` to `origin`. Do not merge it and do not open a pull request.
3. Report both commit hashes as they appear on GitHub.
4. List every file changed on `main` since `9d94995`. If anything beyond the four approved items changed, say so explicitly.

The four approved items were: one `runValidationPipeline` pass per source; the same decision for each source across the review queue, routing review, telemetry and digest; "No Candidate" kept as its own decision; `npm test` and `validate:benchmark` added to `scripts/ops/validate.sh`.

### Task 2. Regenerate the replay from a script

1. Write a replay script and commit it on the experimental branch (for example `src/tools/replayArchivedStage3Runs.ts`). It must read source objects from `audit-evidence/stage3-runs/` and make no network or paid calls.
2. The script writes one machine-generated data file (JSON or CSV), committed on the same branch. One row per source, with these fields:
   - run ID
   - source ID exactly as archived
   - source URL exactly as archived
   - the `validationDecision` recorded in the archive
   - the decision on `main`
   - the decision on the experimental branch
   - for any row whose decision differs between `main` and the branch: which fields changed and the exact text that triggered each change
3. The script must fail if any of these self-checks fail:
   - every source ID digest equals the first 10 hex characters, uppercase, of `sha256(sourceUrl)`
   - every source ID exists in that run's archive
   - the row count per run equals the archive's source count for that run
4. Every table and total in the report must be produced from that data file. No hand-written rows.
5. If a run cannot be replayed, say so and leave it out. Runs `20260826T020143Z` and `20260829T003007Z` are archived in a different layout from the later runs. Do not fill gaps with plausible-looking rows.
6. Where the decision on `main` differs from the archived `validationDecision`, explain why for each case.

### Task 3. Explain the first set of tables

State how the per-source tables in `antigravity_analysis.md` were produced, and why 219 rows carry IDs and URLs that are not in the archive. The operator needs to know whether a script produced them or whether they were written by hand.

### Task 4. Report back

Replace section 2 of `workspace/antigravity_analysis.md` with the regenerated results, and add the answers to Tasks 1 and 3 at the top. Keep the report short: totals per run, the link to the data file on GitHub, and details only for sources whose decision changed.

## On hold until the operator says otherwise

- Merging `exp/connector-intake-refinements`, in whole or in part.
- Any change to role inference (`inferOperatorRole`, `inferBuyerRole`, `ROLE_TERMS`). The five-vertical lexicon proposal is not approved. It repeats the original design with five hardcoded vocabularies instead of one, and the next vertical Aven searches would return "unknown" again. The operator will first supply a labeled list of the 24 sources from run `20261028T222943Z`, and any role fix will be judged against that list.
- Bridging live discovery to the admission gate and decision ledger (Phase 2 of the earlier plan).
- The worker budget cap change (Phase 3).
- Deleting the queue-worker, materializer and offline-sourcing modules (Phase 4).

## Standing rules for reports

- Cite files by repository path and line, not by local `file:///` path.
- Every source ID and URL in a report must come from a file in the repository.
- If something was not run or could not be verified, say so in that sentence.
