# Aven Review Round 2 Response

- **Repository & branch for all file/line citations:** `dgregory3700/goose-worker-aven` on `main` at `0fde501` (`0fde501f52e0dc5fff1b4e689a1b40d6d399a1d8`). Adapter, connector, and validation gate line numbers are unchanged from `aa12d39`.
- **Self-check expectation policy:** **No existing test or self-check expected result was changed.** `npm run build`, `npm test` (27/27), `npm run validate:benchmark` (8/8), `validate:controlled-live-research`, `validate:validation-pipeline`, and `validate:claim-evidence-ingestion` all pass unchanged on `main`.
- **Experimental branch:** `exp/connector-intake-refinements` remains untouched at `695c0ca` as an archived record and will not be merged.

---

## 1. Task 1: Regenerated Section 3 (Run `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z` on `main`)

### 1.1 Root Cause in `src/adapters/sources/AllowlistHttpSourceAdapter.ts` (`main`)

In `src/adapters/sources/AllowlistHttpSourceAdapter.ts:90-111` (`ROLE_TERMS`) and `src/adapters/sources/AllowlistHttpSourceAdapter.ts:1740-1759` (`inferOperatorRole`), operator-role inference checks four government/administrative keyword branches and returns five exact strings:

1. **Honolulu DPP job-bulletin phrases (`:1742-1744`):** if `lower` contains `"permit technician"`, `"permit counter"`, `"files and maintains"`, `"compiles and maintains"`, or `"coordinates and tracks permits"` -> returns `"permit technician, permit counter staff, or administrative permit operator"`.
2. **Account/admin check (`:1746-1748`):** if `matchedRoleTerms` includes `"account holder"` or `"administrator"` -> returns `"account holder, administrator, or reporting owner"`.
3. **Clerical/coordinator check (`:1750-1752`):** if `matchedRoleTerms` includes `"clerk"`, `"technician"`, or `"coordinator"` -> returns `"clerk, technician, coordinator, administrator, or administrative operator"`.
4. **Submitter check (`:1754-1756`):** if `lower` contains `"applicant"`, `"submitter"`, `"preparer"`, `"operator"`, or `"respondent"` -> returns `"applicant, respondent, submitter, preparer, or workflow operator"`.
5. **Fallback (`:1758`):** returns `"unknown operator role; needs direct validation"`.

`"staff"` in `ROLE_TERMS` (`src/adapters/sources/AllowlistHttpSourceAdapter.ts:107`) is not referenced in either `inferBuyerRole` (`:1723-1738`) or `inferOperatorRole` (`:1740-1759`).

Across all 24 archived source-queue objects in `audit-evidence/stage3-runs/OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z/runs/live-source-connector/OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z-SOURCE-CONNECTOR/source-queue/`:

- Every source (`24/24`) has `candidateIntake.likelyOperator = "unknown operator role; needs direct validation"`. Because `src/sourceConnector/liveSourceConnector.ts:350` sets `targetProofObject.operatorOwner = rawSignal.operatorRoleGuess ?? "unknown operator"`, `candidate_intake_gate` (`src/validationPipeline/validators.ts:78-87,367-369`) rejects any string starting with `"unknown"`, so all 24 sources return `"No Candidate"` on `main`.
- `20/24` sources had `matchedRoleTerms.length === 0` on the fetched page (verified from `evidenceNotes[0].weakensClaim` containing `"no buyer/operator role terms were extracted"` at `:1807`, plus `SRC-LIVE-CONNECTOR-023-8114DC68CA`, which hit `isAccessBlockedPage` at `:378-389`).
- Exactly `4/24` sources matched `ROLE_TERMS` entries handled only by `inferBuyerRole` (`:1723-1738`), leaving `inferOperatorRole` at the fallback:
  - `SRC-LIVE-CONNECTOR-005-52C61A1B46` and `SRC-LIVE-CONNECTOR-015-2FF8140F22` matched `"county"`, `"agency"`, or `"department"` (`:1725-1727` -> `"public agency, county department, or compliance owner"`).
  - `SRC-LIVE-CONNECTOR-010-8E3608361C` and `SRC-LIVE-CONNECTOR-021-A783238AED` matched `"employer"`, `"provider"`, `"vendor"`, or `"licensee"` (`:1733-1735` -> `"regulated organization or operating entity"`).

### 1.2 Exact `inferConfidence` Computation (`src/adapters/sources/AllowlistHttpSourceAdapter.ts:1761-1789` on `main`)

`inferConfidence` (`:1761-1780`, capped by `getConfidenceCeiling` at `:1782-1789`) computes:

- Base score: `0.18` (`:1767`)
- Category bonus:
  - `"official_documentation"`: `+0.16` (`:1769`, ceiling `0.6` at `:1783`)
  - `"operator_pain"`, `"public_discussion"`, or `"buyer_evidence"`: `+0.22` (`:1770`, ceiling `0.72` at `:1784`)
  - `"pricing_budget"` or `"competitor_vendor"`: `+0.16` (`:1771`, ceiling `0.68` at `:1786`)
  - `"job_posting"`: `+0.18` (`:1772`, ceiling `0.7` at `:1785`)
  - `"occupational_profile"`: `+0.12` (`:1773`, ceiling `0.58` at `:1787`)
  - `"secondary_commentary"`: `+0.00` (ceiling `0.55` at `:1788`)
- Term bonuses (computed on full cleaned `page.text` at `:392-399`, prior to 900-char summary trimming):
  - `matchedSeedTerms.length > 0`: `+0.08` (`:1774`; for seed `"SaaS workflow evidence"`, `getSeedTerms` at `:1332-1338` yields `["saas", "workflow", "evidence"]`, none of which appear in `WORKFLOW_TERMS` at `:43-88`)
  - `matchedWorkflowTerms.length > 0`: `+0.14` (`:1775`)
  - `matchedRoleTerms.length > 0`: `+0.08` (`:1776`)
- Blocked-page early return: `isAccessBlockedPage` (`:378-389`) bypasses `inferConfidence` and returns `confidence: 0.12`.
- `src/sourceConnector/liveSourceConnector.ts:333` sets `workflowReality: rawSignal.confidence >= 0.5 ? "proven" : "unresolved"`.

Because `createSupportsClaim` (`:1791-1800`) prepends all `matchedSeedTerms` (at most 3 tokens) before `matchedWorkflowTerms` in `.slice(0, 8)`, and `createWeakensClaim` (`:1802-1813`) records whether `matchedWorkflowTerms` and `matchedRoleTerms` were empty on `page.text`, every source's exact fetch-time confidence is deterministically recoverable from the archived queue objects (`24/24` match archived `workflowReality`):

- Of the five sources without workflow terms previously claimed to score `0.40`, only `SRC-LIVE-CONNECTOR-019-BA76DB96E9` (`operator_pain`, `0.18 + 0.22 = 0.40`) scored `0.40`; `SRC-LIVE-CONNECTOR-002-1031638C34` (`competitor_vendor`) scored `0.18 + 0.16 = 0.34`, while `SRC-LIVE-CONNECTOR-004-5BE015C6EA`, `SRC-LIVE-CONNECTOR-008-6407E2BA5E`, and `SRC-LIVE-CONNECTOR-012-31BA6587D5` (`secondary_commentary`) scored `0.18 + 0.00 = 0.18`.
- Missing `+0.08` from `matchedRoleTerms` directly kept `SRC-LIVE-CONNECTOR-001-8C9108525D` and `SRC-LIVE-CONNECTOR-007-4F54E4EBD0` (`competitor_vendor` with workflow terms: `0.18 + 0.16 + 0.14 = 0.48 < 0.50`) below `workflowReality = "proven"`.

| # | Archived `sourceId` | Archived `sourceCategory` / quality | Seed (`+0.08`) | Workflow (`+0.14`) | Role (`+0.08`) | Confidence (`inferConfidence`) | Archived `workflowReality` | Archived `likelyBuyer` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 001 | `SRC-LIVE-CONNECTOR-001-8C9108525D` | `competitor_vendor` / `C` | no | yes | no | `0.18 + 0.16 + 0.14 = 0.48` | `unresolved` | `unknown buyer role; needs direct validation` |
| 002 | `SRC-LIVE-CONNECTOR-002-1031638C34` | `competitor_vendor` / `C` | no | no | no | `0.18 + 0.16 = 0.34` | `unresolved` | `unknown buyer role; needs direct validation` |
| 003 | `SRC-LIVE-CONNECTOR-003-D40628310A` | `public_discussion` / `B` | no | yes | no | `0.18 + 0.22 + 0.14 = 0.54` | `proven` | `unknown buyer role; needs direct validation` |
| 004 | `SRC-LIVE-CONNECTOR-004-5BE015C6EA` | `secondary_commentary` / `E` | no | no | no | `0.18 + 0.00 = 0.18` | `unresolved` | `unknown buyer role; needs direct validation` |
| 005 | `SRC-LIVE-CONNECTOR-005-52C61A1B46` | `operator_pain` / `B` | no | yes | yes | `0.18 + 0.22 + 0.14 + 0.08 = 0.62` | `proven` | `public agency, county department, or compliance owner` |
| 006 | `SRC-LIVE-CONNECTOR-006-46B24CB8CC` | `operator_pain` / `B` | no | yes | no | `0.18 + 0.22 + 0.14 = 0.54` | `proven` | `unknown buyer role; needs direct validation` |
| 007 | `SRC-LIVE-CONNECTOR-007-4F54E4EBD0` | `competitor_vendor` / `C` | no | yes | no | `0.18 + 0.16 + 0.14 = 0.48` | `unresolved` | `unknown buyer role; needs direct validation` |
| 008 | `SRC-LIVE-CONNECTOR-008-6407E2BA5E` | `secondary_commentary` / `E` | no | no | no | `0.18 + 0.00 = 0.18` | `unresolved` | `unknown buyer role; needs direct validation` |
| 009 | `SRC-LIVE-CONNECTOR-009-D5F07008BA` | `public_discussion` / `B` | yes (`workflow`) | yes | no | `0.18 + 0.22 + 0.08 + 0.14 = 0.62` | `proven` | `unknown buyer role; needs direct validation` |
| 010 | `SRC-LIVE-CONNECTOR-010-8E3608361C` | `public_discussion` / `B` | yes (`workflow`) | yes | yes | `0.18 + 0.22 + 0.08 + 0.14 + 0.08 = 0.70` | `proven` | `regulated organization or operating entity` |
| 011 | `SRC-LIVE-CONNECTOR-011-08623C1BC3` | `secondary_commentary` / `D` | no | yes | no | `0.18 + 0.00 + 0.14 = 0.32` | `unresolved` | `unknown buyer role; needs direct validation` |
| 012 | `SRC-LIVE-CONNECTOR-012-31BA6587D5` | `secondary_commentary` / `D` | no | no | no | `0.18 + 0.00 = 0.18` | `unresolved` | `unknown buyer role; needs direct validation` |
| 013 | `SRC-LIVE-CONNECTOR-013-17D3CE419F` | `public_discussion` / `B` | no | yes | no | `0.18 + 0.22 + 0.14 = 0.54` | `proven` | `unknown buyer role; needs direct validation` |
| 014 | `SRC-LIVE-CONNECTOR-014-43A1016733` | `secondary_commentary` / `D` | no | yes | no | `0.18 + 0.00 + 0.14 = 0.32` | `unresolved` | `unknown buyer role; needs direct validation` |
| 015 | `SRC-LIVE-CONNECTOR-015-2FF8140F22` | `operator_pain` / `B` | no | yes | yes | `0.18 + 0.22 + 0.14 + 0.08 = 0.62` | `proven` | `public agency, county department, or compliance owner` |
| 016 | `SRC-LIVE-CONNECTOR-016-13E84E92DC` | `operator_pain` / `B` | no | yes | no | `0.18 + 0.22 + 0.14 = 0.54` | `proven` | `unknown buyer role; needs direct validation` |
| 017 | `SRC-LIVE-CONNECTOR-017-DBF01F66DC` | `secondary_commentary` / `D` | no | yes | no | `0.18 + 0.00 + 0.14 = 0.32` | `unresolved` | `unknown buyer role; needs direct validation` |
| 018 | `SRC-LIVE-CONNECTOR-018-AF446C24FC` | `secondary_commentary` / `D` | no | yes | no | `0.18 + 0.00 + 0.14 = 0.32` | `unresolved` | `unknown buyer role; needs direct validation` |
| 019 | `SRC-LIVE-CONNECTOR-019-BA76DB96E9` | `operator_pain` / `B` | no | no | no | `0.18 + 0.22 = 0.40` | `unresolved` | `unknown buyer role; needs direct validation` |
| 020 | `SRC-LIVE-CONNECTOR-020-80F2AF689F` | `secondary_commentary` / `E` | no | yes | no | `0.18 + 0.00 + 0.14 = 0.32` | `unresolved` | `unknown buyer role; needs direct validation` |
| 021 | `SRC-LIVE-CONNECTOR-021-A783238AED` | `operator_pain` / `B` | no | yes | yes | `0.18 + 0.22 + 0.14 + 0.08 = 0.62` | `proven` | `regulated organization or operating entity` |
| 022 | `SRC-LIVE-CONNECTOR-022-DAEFD2F39F` | `public_discussion` / `B` | yes (`workflow`) | yes | no | `0.18 + 0.22 + 0.08 + 0.14 = 0.62` | `proven` | `unknown buyer role; needs direct validation` |
| 023 | `SRC-LIVE-CONNECTOR-023-8114DC68CA` | `secondary_commentary` / `E` | no | no | no | `0.12` (early return in `isAccessBlockedPage`, `:378-389`) | `unresolved` | `unknown buyer role; needs direct validation` |
| 024 | `SRC-LIVE-CONNECTOR-024-69024BB54D` | `operator_pain` / `B` | no | yes | no | `0.18 + 0.22 + 0.14 = 0.54` | `proven` | `unknown buyer role; needs direct validation` |

---

## 2. Task 2: Stage-3 Label Scoring Tool on `main`

- **Commit on `main`:** `0fde501` (`0fde501f52e0dc5fff1b4e689a1b40d6d399a1d8`)
- **Files added/updated:**
  - `src/tools/scoreArchivedStage3Runs.ts` (offline scorer with all three archive self-checks across all 9 Stage-3 runs / 223 sources, label-to-archive hash/URL/count verification, and decision-unity check across `runValidationPipeline` and `routeCandidateHypothesisProofPack`)
  - `benchmark/labels/aven_yardstick_20261028.json` (unchanged copy, git blob `9571c0f74b7658dfbd004608b223f8e60a0c2ee6`)
  - `benchmark/labels/aven_holdout_20261021.json` (unchanged copy, git blob `02e75b1d2ab46cd9c6a8e0bffa7d4225d9d30103`)
  - `package.json` (`npm run score:stage3-labels`)

### 2.1 Verbatim Tool Output on `main` (`npx tsx src/tools/scoreArchivedStage3Runs.ts`)

```text
Archive self-checks passed: 9 runs, 223 sources verified (allSourceIdHashesVerified=true, allSourceIdsExistInArchive=true, allRunSourceCountsMatchArchive=true).

Run: OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z (benchmark/labels/aven_yardstick_20261028.json, 24 sources)
  surface reaching Park        : 0 / 7
  do_not_surface reaching Park : 0 / 12
  too_thin reaching Park       : 0 / 3
  borderline reaching Park     : 0 / 2
  overall decisions            : {"No Candidate":24,"Reject":0,"Park":0,"Advance":0}
  operator roles on surface sources:
    [001] SRC-LIVE-CONNECTOR-001-8C9108525D (No Candidate): assigned="unknown operator role; needs direct validation" | expected="self-managing landlord"
    [003] SRC-LIVE-CONNECTOR-003-D40628310A (No Candidate): assigned="unknown operator role; needs direct validation" | expected="Shopify merchant doing own bookkeeping"
    [006] SRC-LIVE-CONNECTOR-006-46B24CB8CC (No Candidate): assigned="unknown operator role; needs direct validation" | expected="landlord or small property manager"
    [009] SRC-LIVE-CONNECTOR-009-D5F07008BA (No Candidate): assigned="unknown operator role; needs direct validation" | expected="Shopify merchant"
    [013] SRC-LIVE-CONNECTOR-013-17D3CE419F (No Candidate): assigned="unknown operator role; needs direct validation" | expected="Shopify merchant doing own bookkeeping"
    [017] SRC-LIVE-CONNECTOR-017-DBF01F66DC (No Candidate): assigned="unknown operator role; needs direct validation" | expected="self-managing landlord"
    [022] SRC-LIVE-CONNECTOR-022-DAEFD2F39F (No Candidate): assigned="unknown operator role; needs direct validation" | expected="Shopify merchant or inventory/purchasing staff"

Run: OE-CONTROLLED-LIVE-RESEARCH-20261021T134618Z (benchmark/labels/aven_holdout_20261021.json, 24 sources)
  surface reaching Park        : 0 / 0
  do_not_surface reaching Park : 7 / 19
  too_thin reaching Park       : 0 / 5
  overall decisions            : {"No Candidate":17,"Reject":0,"Park":7,"Advance":0}
  operator roles on surface sources: (none — 0 surface sources in this label file)
```

This reproduces the `main` baseline row (`0 / 7` yardstick `surface`, `0 / 12` yardstick `do_not_surface`, `7 / 19` holdout `do_not_surface`).

### 2.2 Limit of Offline Archive Scoring (Retained Excerpts vs. Full Page Text)

Each archived queue object under `audit-evidence/stage3-runs/` stores `sourceUrl`, `title`, and the passage-selected `summary` / `claimEvidence[0].excerpt` trimmed to at most 900 characters (`src/adapters/sources/AllowlistHttpSourceAdapter.ts:1345-1365`), **not** the raw HTML or the full cleaned `page.text` (up to 12,000 characters at `:326-333`).

- **What the archive CAN test for a categorization change:**
  1. Rules based on URL host, URL path segments, or `title` (e.g., discussion thread paths vs. `/blog/`, `/solutions/`, `/guide/`, `/newsletters/`, `/issues/`, or `"How to ..."` titles).
  2. Rules inspecting prose structure, first-person testimony, sales copy, or extraction thinness/UI chrome **within the retained 900-char excerpt**.
  3. All downstream effects of category/quality changes on `sourceGrade` (`src/sourceConnector/liveSourceConnector.ts:332`), `inferConfidence` (`:1761-1789`), `inferClaimResult` (`:445-482`), and `runValidationPipeline` (`src/validationPipeline/runValidationPipeline.ts:21-65`).
- **What the archive CANNOT test:**
  1. Whether keyword or phrase checks in `inferSourceCategory` (or `inferOperatorRole` / `inferConfidence`) would match on the unretained portion of the 12,000-character fetched page — such as sidebar/footer chrome (`"support"`, `"software"`, `"product"` at `:1569`), bylines, or later replies not chosen by `createSummary`. For example, in `OE-CONTROLLED-LIVE-RESEARCH-20261021T134618Z`, all 7 archived `Park` sources matched their `ROLE_TERMS` keyword outside the retained `summary`; in `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z`, `002-1031638C34` matched `competitor_vendor` terms and `019-BA76DB96E9` matched `operator_pain` terms outside their 53-char and 43-char retained excerpts.
  2. Rules that depend on HTML DOM structure, element attributes, or author/post boundaries discarded before `PageSnapshot`.

---

## 3. Task 3: Design Proposal (No Code Changes)

### 3.1 Extraction Floor Prerequisite (Thin and Blocked Pages)

Before scoring role, category, or contradictions, `findIntrinsicUnusableExtractionReason` (`src/sourceConnector/liveSourceConnector.ts:259-272`) should quarantine not only explicit block messages, but also (a) retained excerpts with fewer than **120 characters** (or fewer than **20 words**) of prose after stripping collapsed HTML separator runs (`. .`) and standalone UI action labels (`Like`, `Reply`, `Follow`, `Share`, timestamps), and (b) excerpts where collapsed separators/UI labels exceed 35% of tokens. When `EXTRACTION-INTEGRITY` notes are present, `source_quality_gate` / `candidate_decision` (`src/validationPipeline/validators.ts:14-17,272-273`) should surface that extraction diagnostic directly in `reasons` rather than emitting only `"Source-quality gate refused candidate intake."`.

### 3.2 Part 1 — Operator Role Inference (`inferOperatorRole` & `inferBuyerRole`)

- **Rule in plain words:**
  1. Extract the operator role only on pages classified as practitioner testimony / operational workflow evidence (never on vendor marketing, editorial newsletters, or software bug reports).
  2. Match **syntactic role frames** in the title and text rather than industry word lists:
     - **First-person self-identification:** `"I'm a / I am a / As a / We are <noun phrase>"` or `"for <noun phrase> who <manage/run/operate/handle> their own <object>"`, filtering out conversational non-role nouns (`newbie`, `beginner`, `member`, `user`, `customer`, `person`) unless paired with an operational task.
     - **Peer-audience vocatives & ownership clauses:** `"<noun phrase> owners —"`, `"fellow <noun phrase>"`, or `"managing / tracking / handling <workflow> alongside/for your <business/asset noun>"`.
     - **Functional actor-action fallback:** When a post describes recurring first- or second-person operational handling of business records without naming a job title (e.g., `"I have to manually enter ... into <System> ... running a report"` or `"goods received in <System A> ... create a bill in <System B>"`), synthesize `"<workflow/artifact> operator"` from the verb + business object (e.g., `"operator performing recurring bookkeeping/accounting entry"` or `"operator reconciling purchase orders and supplier bills"`).
  3. Restrict `inferBuyerRole`'s `"public agency, county department, or compliance owner"` label (`:1725-1727`) to pages with explicit public-sector markers (`.gov`, `city council`, `municipal`, `public agency`, `county`), removing bare `"department"` and `"agency"` triggers that mislabel private businesses.
- **Why it is general:** Every vertical uses the same English grammar to state who is speaking (`"I'm a ..."`), whom a thread addresses (`"<X> owners"`, `"for your <X>"`), or what recurring clerical action the speaker performs (`"I have to manually enter <X>"`). A veterinary clinic manager, customs broker, or HOA treasurer is identified by the same clause structures without adding any industry vocabulary.
- **Expected score (combined with Parts 2–3):**
  - **Yardstick (`20261028`):** `7 / 7` `surface` items receive a private-sector operator role matching `expectedOperatorRole` in meaning (`001`: small-time landlord; `003`: store operator managing bookkeeping; `006`: property operator tracking maintenance requests; `009`: store owner; `013`: operator performing monthly bookkeeping entry; `017`: real estate investor privately managing own properties; `022`: operator reconciling purchase orders and supplier bills); `0` receive government labels.
  - **Holdout (`20261021`):** `0 / 19` `do_not_surface` items receive an operator role or reach `Park`.
- **What it could get wrong:**
  - Dual-credential self-descriptions (e.g., `001`'s `"small-time landlord and software engineer"`) can retain the secondary credential unless anchored to the sentence's operational object (`"tenant maintenance requests"`).
  - Anonymous passive-voice threads describing a real workflow without `"I/we/your"` or a peer vocative could miss syntactic frames unless the functional verb+artifact fallback fires, or conversely an overly broad verb+artifact fallback could label a casual consumer complaint if Part 2's categorization gate failed to filter it first.

### 3.3 Part 2 — Categorization (`inferSourceCategory` & `inferEvidenceQuality`)

- **Rule in plain words:**
  1. **Classify discourse genre before testing pain keywords:**
     - **Vendor / product / template pages -> `competitor_vendor` (quality `C`, `sourceGrade = false`):** Any non-discussion URL (not `/forums/`, `/forum/`, `/threads/`, `/t/`, `/c/`, `/discuss/`, `/discussions/`, `/questions/`, `/community/`) on paths such as `/solutions/`, `/features/`, `/products/`, `/industries/`, `/templates/`, `/guide/`, `/r/`, `/pricing`, or carrying second-person product instructional/marketing framing (`"How to <verb>"` tutorials, `"let the software handle the rest"`, `"The best software should let..."`, `"automates three-way matching"`, demo/trial CTAs). Remove the broad `:1569-1571` page-text fallback (`"support"`, `"software"`, `"product"`) on discussion URLs so user forum posts are not misclassified as `competitor_vendor` from page chrome or incidental mentions.
     - **Editorial, trade press, newsletters, & software repo bug trackers -> `secondary_commentary` (quality `D`/`E`, `sourceGrade = false`):** Non-thread article/newsletter paths and markers (`/blog/`, `/posts/`, `/newsletters/`, `-article`, `/news/`, `coding-alert`, journalistic attribution `"according to [Name]"`, advisory imperatives `"Follow This Advice"`, `"Ways to Avoid"`, `"What Is ...?"` explainers) and software repository issue/PR pages (`/issues/`, `/pull/`, repro/stack-trace/component-bug reports).
  2. **Require compound operational pain for source-grade testimony:**
     - Demote plain `public_discussion` without compound operational pain from quality `B` to quality `C` (`sourceGrade = false`), so casual forum threads never pass `Source Quality Gate` merely for being on a forum.
     - Assign `operator_pain` (quality `B`, `sourceGrade = true`) **only** when a discussion/testimony source pairs **(a) a recurring operational artifact or cadence** (spreadsheets, paper/notes, reports/exports, invoices/bills/orders/claims/requests, or `every week/month/day`, `repetitive`) with **(b) active multi-step manual friction** (cross-system manual re-entry/double work, reconciliation across tools, unstructured multi-channel tracking chaos, or explicit manual-step sequences), **excluding** single-tool configuration quirks where an existing feature already works (`"works fine as long as..."`) and hypothetical setups with explicit free-only constraints and no current manual burden (`"free tool"`, `"free is preferred"` with no pain).
- **Why it is general:** It separates **who is speaking and why** (first-person peer discussion vs. vendor content marketing, trade journalism, template directories, and software bug trackers) using universal web/discourse structure rather than domain blocklists, and defines pain as compound recurring clerical/reconciliation friction rather than single words like `"manual"` or `"staff"`.
- **Expected score (combined with Parts 1 & 3, checked once against holdout without tuning):**
  - **Yardstick (`20261028`):** `7 / 7` `surface` reach `Park`; `0 / 12` `do_not_surface` reach `Park`; `0 / 3` `too_thin` reach `Park` (all 3 carry explicit extraction reasons: `002` 53-char floor, `008` UI-chrome/32-char prose floor, `023` HTTP 403 block). For the 2 `borderline` items, `007` stays out (`"Free is preferred"`, no pain) while `010` may either stay out (single-report accuracy issue) or `Park` depending on whether broken third-party reporting counts as cross-tool friction; both outcomes satisfy `borderline`.
  - **Holdout (`20261021`):** `0 / 19` `do_not_surface` reach `Park` (all 13 vendor/service pages and 1 template page are classified `competitor_vendor`/`secondary_commentary` via `/blog/`, `/guide/`, `/industries/`, `/solutions/`, `/posts/`, `/templates/`, and non-discussion `"How to..."` / explainer framing; all 4 GitHub `/issues/` tickets are `secondary_commentary`; `001` `.gov` fails `official_system_absorption_check` and lacks an operator role). `0 / 5` `too_thin` reach `Park`, and all 5 carry extraction reasons (`004, 005, 014, 020` HTTP block markers; `008` 43-char GitHub refusal message caught by the 120-char floor).
- **What it could get wrong:**
  - A genuine operator writing a first-person post mortem on a personal `/blog/` URL would be classified as `secondary_commentary` and missed.
  - Vendor astroturfing inside a real forum thread (written in first-person without an obvious product link in the excerpt) could reach `Park` for human review.
  - A real B2B operations team tracking internal clerical pain in a public GitHub `/issues/` tracker would be filtered out alongside software bug reports.

### 3.4 Part 3 — Work-Removal Contradiction Matching (`inferClaimResult("work_removal")`)

- **Rule in plain words:**
  - In `src/sourceConnector/liveSourceConnector.ts:465-470`, stop marking `work_removal` as `"contradicted"` on bare substring matches of `"re-enter"`, `"duplicates"`, `"relocates"`, or `"additional data entry"`.
  - Mark `work_removal = "contradicted"` **only** when non-negated evidence states that a **proposed or existing tool/intervention itself** still requires manual re-entry, duplicates portal work, or shifts entry to another step — e.g., `"merely relocates"`, `"only duplicates"`, `"must still re-enter"`, `"still requires manual entry"`, or `"<tool/system> requires operators to re-enter"` — and **never** when the term appears in a negated/goal clause (`"don't have to re-enter"`, `"without manually entering"`, `"rather than re-entering"`, `"instead of re-entering"`, `"avoid/eliminate/reduce re-entry"`) or describes the baseline manual problem being replaced.
  - Do **not** loosen `work_removal = "proven"`: removing a false contradiction leaves `work_removal` at `"unproven"` (`warn`), routing eligible sources to `Park` without making `Advance` easier to reach.
- **Why it is general:** In English, every request for automation names the manual action it wants to stop doing (`"without X"`, `"rather than X"`, `"so we don't have to X"`). Distinguishing baseline pain / negated goals from tool-caused relocation is a polarity and subject-attribution rule, independent of any site or vertical, and preserves existing self-checks (including `src/tools/validateClaimEvidenceIngestion.ts:71-79`: `"This merely relocates work: operators must re-enter data into a second system."`).
- **Expected score:**
  - **Yardstick (`20261028`):** Unblocks `SRC-LIVE-CONNECTOR-022-DAEFD2F39F` from `Reject` (`work_removal: contradicted`) to `Park` (`work_removal: unproven`), bringing `surface` from `6 / 7` to `7 / 7` while keeping `do_not_surface` at `0 / 12`.
  - **Holdout (`20261021`):** `0 / 19` `do_not_surface` reach `Park` (noise pages are already stopped at `Source Quality Gate` by Part 2).
- **What it could get wrong:**
  - Nuanced thread replies saying an integration works halfway except for edge cases (e.g., *"the sync app pushes headers, though line-item taxes still need manual adjustment in Xero"*) will stay `unproven` (`Park`) rather than `contradicted` (`Reject`), leaving that distinction to human review.
  - Unusual double-negation or long-distance clauses separating the negation cue from `"re-enter"` beyond the local clause window could misclassify polarity.

### 3.5 Observations on the Queries Across the Two Runs (No Changes Proposed)

Both runs recorded `seed: "SaaS workflow evidence"` and `origin: "operator_supplied"`, executing explicit CLI query lists rather than generated defaults:

- **`OE-CONTROLLED-LIVE-RESEARCH-20261021T134618Z` (6 queries, holdout):**
  - `[0]` `re-enter the same data manually every week workaround small business forum`
  - `[1]` `accounts payable invoices re-keyed by hand into accounting system complaint`
  - `[2]` `property management maintenance requests tracked in spreadsheets workaround`
  - `[3]` `freight broker manually typing bill of lading data into TMS complaint`
  - `[4]` `site:github.com issues re-enter information again after rejection workaround`
  - `[5]` `clinic staff resubmit denied claim manually same information again`
- **`OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z` (5 queries, yardstick):**
  - `[0]` `site:quickbooks.intuit.com manually re-enter invoices every month workaround -blog`
  - `[1]` `site:biggerpockets.com/forums maintenance requests spreadsheet tenants workaround`
  - `[2]` `site:thetruckersreport.com dispatcher manually typing load information every day`
  - `[3]` `site:community.shopify.com manually re-enter orders into accounting every week`
  - `[4]` `site:aapc.com denied claims resubmit manually same information again`

**Observations:**
1. **Open-web pain phrases attract vendor SEO:** Five of six queries in `20261021` were unscoped web searches for pain phrases (`"re-enter the same data manually"`, `"invoices re-keyed"`, `"tracked in spreadsheets"`, `"resubmit denied claim"`). Because SaaS vendors title their content marketing with those exact phrases, 13 of 24 fetched URLs were vendor/agency blogs and landing pages, while the 2 Reddit threads that surfaced (`004`, `005`) returned HTTP blocks.
2. **Subpath forum scoping vs. root-domain scoping:** In `20261028`, `site:biggerpockets.com/forums` and `site:community.shopify.com` returned 10 user discussion threads out of 10 (yielding all 7 `surface` leads). Root-domain `site:aapc.com` (rather than `site:aapc.com/discuss`) returned 3 `/codes/coding-newsletters/` articles and 1 blocked `/blog/` post alongside 1 forum thread, and `site:quickbooks.intuit.com ... -blog` still admitted a `/r/invoicing/how-to-...` product guide (`016`).
3. **Actor mismatch within a forum:** `site:thetruckersreport.com dispatcher manually typing load information every day` searched a driver-centric board where `"dispatcher"` appears primarily in driver complaints about dispatchers (`004, 012, 014, 018, 019`) rather than back-office dispatchers describing load entry.
4. **Unscoped `site:github.com`:** `site:github.com issues re-enter information again after rejection workaround` in `20261021` matched generic UI/retry/transcript words in unrelated software engineering repositories (`003, 008, 009, 010, 011`).

---

## 4. Task 4: SSH Host Audit and Read-Only `goose-worker-aven` Status

### 4.1 Operating Rules Acknowledged
- Aven work uses **only** the local checkout (`C:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven`) and SSH host `goose-worker-aven` (`ubuntu@3.139.138.125`, per `USER_MANUAL.md:118` and `.agents/skills/session-closeout/SKILL.md:23,89`).
- `chessa-ec2-chatgpt` (`3.129.14.7`) belongs to Chessa and will never be connected to, read from, written to, or executed against for Aven work.

### 4.2 Session-by-Session Audit Since 2026-11-23 (Verified from `.system_generated/logs/transcript_full.jsonl`)

| Session / Conversation ID | Time Window (UTC) | Where Work Was Done | Commands Run on `chessa-ec2-chatgpt` (`3.129.14.7`) |
| --- | --- | --- | --- |
| `8ad88b91-bb48-4eff-afe3-9e739808889a` (2026-11-23..24 audit, Phase 1, replay & closeout) | `2026-11-23T11:55Z` – `2026-11-24T19:36Z` | Local checkout + 2 closeout SSH calls to `goose-worker-aven` (`transcript_full.jsonl` lines 331 and 345: status check and `git pull --ff-only` to `74c2cf0`) | **None (`0`)** |
| `b4db66f9-3204-473e-8fd0-259972b092a2` (GitHub auth scope check) | `2026-11-24T02:48:21Z` – `2026-11-24T02:48:31Z` | Local only (`gh auth status`; `0` SSH commands) | **None (`0`)** |
| `950431eb-3bd5-4e93-858f-6d34e7f855aa` (current session, 2026-11-24) | `2026-11-24T19:39Z` – present | Local checkout + read-only preflight SSH check on `goose-worker-aven` (checked local `~/.ssh/config` and local transcripts only; no pull/start/stop on EC2) | **None (`0`)** |

### 4.3 Read-Only Status of `goose-worker-aven` (Observed `2026-11-24 19:41:48 UTC`)

- **Checkout & commit:** `/home/ubuntu/goose-worker-aven` on branch `main` at `74c2cf0` (`74c2cf06799148b3e45801cfea7018c528c097d1`, `docs(sessions): handoff 2026-11-24 and next-session SSP for claude_review follow-up`), clean working tree. Per Task 4.4, **no `git pull` was run on `goose-worker-aven`** after pushing Task 2 (`0fde501`) to `origin/main`, awaiting operator approval.
- **Aven research process:** **None running** (`No Aven research process found.`; `pm2 jlist` is `[]`).
- **Lockdown & ledger:** `.env` has `OE_LIVE_RESEARCH_ENABLED=false`, `OE_LIVE_RESEARCH_KILL_SWITCH=true`, `OE_LIVE_SEARCH_PROVIDER=brave`, `OE_LIVE_MODEL_PROVIDER=none`; `status/live-research-budget-ledger.json` remains at `dayKey: "2026-10-28"`, `spendUsd: 0.025`.

---

## 5. Task 5: Global Research Skill for Exa and Tavily

- **Skill path (global, applies across all projects on this machine):** `~/.gemini/config/skills/external-web-research/SKILL.md` (`C:/Users/dgreg/.gemini/config/skills/external-web-research/SKILL.md`, discovered under Antigravity's global customization root `~/.gemini/config/`).
- **Provider note for the operator (suggestion only; nothing wired into Aven):** Because `tavily_search` enforces `include_domains` and `exclude_domains` as hard filters while Brave `-blog` still admitted vendor paths like `/r/invoicing/...` and unscoped pain queries returned mostly SEO blogs, Tavily (for hard forum-domain allowlisting/exclusion) or Exa (for semantic descriptions of first-person practitioner threads) may be worth a future operator-authorized comparison against Brave for Aven's search step.

### Full Text of `~/.gemini/config/skills/external-web-research/SKILL.md`

````markdown
---
name: external-web-research
description: >-
  Guides external web research using both Exa and Tavily MCP tools. Activate this
  skill any time information from outside the repository is needed, including
  library or API documentation, error messages, how a website structures its pages,
  facts about a vendor or market, or verifying a claim before repeating it.
---

# External Web Research (Exa + Tavily)

Use both connected research MCP servers whenever a task requires information from outside the local repository.

## 1. When This Skill Applies

Activate this workflow whenever outside information is needed, including:
- Library, SDK, CLI, or API documentation
- Investigating an unfamiliar error message or protocol behavior
- Checking how a website structures its URLs, threads, or pages
- Looking up facts about a company, vendor, product, or market
- Verifying any external claim or figure before repeating it

## 2. Connected MCP Tools (Exact Names)

Invoke tools through `call_mcp_tool` (inspect `<appDataDir>/mcp/<ServerName>/<ToolName>.json` first if unsure of parameters):

- **Server `exa`**
  - `web_search_exa`: Semantic search where `query` describes the ideal page rather than bare keywords; supports `category:company` and `category:people` focus prefixes and `numResults`.
  - `web_fetch_exa`: Reads full page content as clean markdown for known `urls` (`maxCharacters`).
  - `agent_run`: Multi-step structured research or entity enrichment (`query` or `runId`, `effort`, `outputSchema`, `input.data`, `input.exclusion`, `budget`).
- **Server `tavily_tavily`**
  - `tavily_search`: Keyword/phrase search returning snippets and URLs; supports hard domain filters (`include_domains`, `exclude_domains`), `exact_match`, `search_depth`, `max_results`, and publish-date filters.
  - `tavily_extract`: Extracts full page text/markdown from `urls` (`extract_depth`: `basic` or `advanced`, optional `query`).
  - `tavily_crawl`: Crawls pages from a root `url` (`max_depth`, `max_breadth`, `limit`, `select_domains`, `select_paths`, `instructions`).
  - `tavily_map`: Lists discovered URLs starting from a root `url`.
  - `tavily_research`: Multi-source cited research report that lists sources and gaps (`input`, `model`: `mini`, `pro`, `auto`).

## 3. Use Each Tool for What It Does Best

Based on operator evaluation (2026-11-20):

- **Exa (`exa`)**
  - Strongest at locating a specific or obscure page when you describe the page you want (`web_search_exa`), and at company lookups (`category:company`).
  - `agent_run` returns structured outputs with a source citation per field when given an `outputSchema`.
  - Treat `"exclude X"` in an Exa query or `input.exclusion` in `agent_run` as a **preference, not a hard filter**; verify exclusions yourself in the returned URLs.
- **Tavily (`tavily_tavily`)**
  - Strongest at producing a cited research summary that explicitly lists its sources and remaining gaps (`tavily_research`).
  - `include_domains` and `exclude_domains` on `tavily_search` are **hard filters**, making Tavily the right choice for forum-only or single-site searches.
  - Can extract (`tavily_extract`), map (`tavily_map`), and crawl (`tavily_crawl`) site pages.
  - **Extraction rule for figures/tables:** When extracting a page to verify a specific figure, table, list, or exact clause, call `tavily_extract` **without** `query` (leave `query` empty), because supplying a `query` reranks/trims chunks and can cut out the figure you need.

## 4. Cross-Check Decisions Across Both Tools

- For any fact, market claim, or technical conclusion that a decision will rest on, **run both Exa and Tavily and compare the results**.
- Where they disagree, state the disagreement plainly and explain which primary source you trust and why.

## 5. Primary-Source Verification and Citations

- **A search snippet is not a source.** Always open/extract the underlying page (`web_fetch_exa` or `tavily_extract`) and confirm the statement in context before citing it.
- Cite the canonical **URL** and the **date read (`YYYY-MM-DD`)** for every external fact.
- State plainly in the same passage anything you could not confirm from primary page text.

## 6. Cost Discipline

- Both Exa and Tavily are paid APIs.
- Start with ordinary search (`web_search_exa`, `tavily_search`) and targeted page extraction (`web_fetch_exa`, `tavily_extract`).
- Escalate to `tavily_research` or Exa `agent_run` **only** when ordinary search and page reads have not answered the question.

## 7. Secret Hygiene

- Never put an API key, token, or credential in any file, commit, report, or chat message.

## 8. Separation from Aven's Evidence Pipeline

- Exa and Tavily are strictly for the agent's own research.
- **Nothing found through Exa or Tavily may enter Aven's source queue, runs, archive, or validation/routing decisions.** Aven's evidence comes exclusively through her own pipeline and budget controls.
- Do **not** wire Exa or Tavily into Aven's codebase unless explicitly instructed by the operator. If either provider appears worth evaluating for Aven's search adapter, propose it in prose for the operator's decision.
````
