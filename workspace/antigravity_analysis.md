### 1. Approved Changes Committed on `main` (`aa12d39`)

All four approved items are implemented and committed on `main` (`aa12d39`):

1. **Single `runValidationPipeline` pass per source:**
   - [types.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/engineOperatingLoop/types.ts#L53-L61) and [runEngineOperatingLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/engineOperatingLoop/runEngineOperatingLoop.ts#L64-L115) now retain `validationResults: ValidationPipelineResult[]` alongside `sourceRuns`.
   - [productOperatingLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/operations/productOperatingLoop.ts#L106-L115) reuses `engineLoop.validationResults[index]` instead of running `runValidationPipeline` a second time.
2. **Unified decision across operator review queue, routing review, telemetry, and digest + distinct `"No Candidate"` decision:**
   - [candidateHypothesis.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/sourcing/candidateHypothesis.ts#L58-L67) preserves `"No Candidate"` when `validation.decision === "No Candidate"` instead of forcing source-grade `"No Candidate"` records into `"Park"` or `"Reject"`.
   - [opportunityRouting.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/sourcing/opportunityRouting.ts#L8-L148) adds `"No Candidate"` to `AutonomousRouteDecision`, tracks `noCandidateCount`, maps `validationDecision: "No Candidate"` directly to `routeDecision: "No Candidate"`, and enforces `route.routeDecision === route.validationDecision` in `validateAutonomousRoutingReviewRun`.
   - [discoveryEffectivenessTelemetry.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/liveResearch/discoveryEffectivenessTelemetry.ts#L28-L320) widens `routeDecision` to `ValidationDecision | null` and asserts `source.validationDecision === source.routeDecision` in `validateDiscoveryEffectivenessTelemetryRun`.
   - [generateOperatorReviewDigest.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/generateOperatorReviewDigest.ts#L22-L202) adds `decisionCounts: { noCandidate, reject, park, advance }` and renders `No Candidate / Reject / Park / Advance` consistently across JSON and Markdown digests.
   - Updated summary formatters and self-checks in [controlledLiveResearchLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/liveResearch/controlledLiveResearchLoop.ts#L455-L459), [runControlledLiveResearchLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/runControlledLiveResearchLoop.ts#L76-L80), [runControlledLiveResearchWorker.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/runControlledLiveResearchWorker.ts#L44-L48), [runProductOperatingLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/runProductOperatingLoop.ts#L23-L27), [validateAutonomousRoutingReview.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/validateAutonomousRoutingReview.ts#L44-L108), and [validateProductOperatingLoop.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/tools/validateProductOperatingLoop.ts#L31-L78).
3. **Validation script updated:**
   - [validate.sh](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/scripts/ops/validate.sh#L33-L35) now runs `run_script_if_present "test"` and `run_script_if_present "validate:benchmark"`.
   - Verified on `main`: `npm run build` (clean), `npm test` (27/27 pass), `npm run validate:benchmark` (8/8 pass), plus `validateProductOperatingLoop`, `validateAutonomousRoutingReview`, and `validateControlledLiveResearchLoop`.

---

### 2. Separate Branch `exp/connector-intake-refinements` (`8547332`, Unmerged) & Offline Replay of Archived Stage-3 Runs

The five unapproved changes are committed solely on branch `exp/connector-intake-refinements` (`8547332`) and **not** merged into `main`:
- [liveSourceConnector.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/sourceConnector/liveSourceConnector.ts):
  1. `workRemoval`, `buyerPackageProof` (from `buyer_evidence`), and `packageReality` (from `package_reality`) derived from `inferClaimResult` via `toValidationProofState`.
  2. `pricing_budget` and `competitor_vendor` included in `sourceGrade` (and assigned evidence quality `"B"` in [AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L1687-L1699)).
  3. `operatorOwner` falls back to `likelyBuyer` (`rawSignal.buyerRoleGuess`) when `operatorRoleGuess` is unknown (mirrored in `candidate_intake_gate` in [validators.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/validationPipeline/validators.ts#L76-L81)).
  4. `createBoundedWorkflowIdentity` infers `industries: [seed.trim()]` when `"permit"` / `"contractor"` are absent.
  5. `inferOfficialSystemSignals` triggers only when `official_documentation` text contains explicit absorption phrases (`"portal-only"`, `"fully absorbed"`, `"official system handles"`, `"built-in submission"`, `"single official portal"`).

> **Note on Archived Run Count:** There are **9** archived run directories under `audit-evidence/stage3-runs/` (8 runs from `20260828T220640Z` through `20261028T222943Z`, plus the earliest run `20260826T020143Z`). All 9 runs (223 total source records) were replayed offline below.
>
> **Headline Takeaway from the Replay:** Across all 223 sources, **0 reach `Advance`** and **81 newly move to `Park`**. Combining all five experimental changes creates significant false-positive promotions to `Park` on generic `.gov` procurement handbooks, contract awards, and even garbled PDF glyph dumps (`001-5D6399B840`, `004-7ABCE2D07E`), because:
> - generic words like `"agency"`, `"department"`, `"county"`, `"vendor"`, or `"provider"` populate `likelyBuyer`, which the fallback promotes into `operatorOwner`;
> - seed fallback populates `industries`; and
> - single keywords like `"contract"`, `"procurement"`, `"form"`, or `"schedule"` in `inferClaimResult` mark `bounded_workflow_identity`, `buyer_evidence`, or `package_reality` as `"proven"`.

---

#### Run 1: `OE-CONTROLLED-LIVE-RESEARCH-20260826T020143Z`
- **Before (`main`):** `{"No Candidate":17,"Reject":4,"Park":4,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":14,"Reject":0,"Park":11,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **7** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-5D6399B840` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/1984-03.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-79F65F892D` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Guidelines-To-Complete-Flood-Hazard-Assessment-Report-rev.9.25.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-003-270341CE1E` | https://www.honolulu.gov/dpp/permitting/building-permits/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-004-9BBFF3FE13` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2026/05/dpp_permit_center_appointment_system.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-005-221DC0E8FC` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Check-List-Residential-2013.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-006-78F726C3EA` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Format-Checklist-2021.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-007-FB02904BD1` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/Solar-photovoltaic-checklist.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-008-1884276FE4` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/prescriptive_pv_recovery_system.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-009-3D4A582C0E` | https://www.governmentjobs.com/careers/honolulu/jobs/2491292-0/building-permit-review-system-assistant | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-010-FF090425F5` | https://www.governmentjobs.com/careers/honolulu/classspecs/763638 | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-011-7C1D728F41` | https://www.honolulu.gov/rep/site/dpp/dpp_docs/plan-review-fee-when-switching-form-opt-to-building-permit.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E52C66E0A` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-279712/31198.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-966C5F8EE1` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-317072/RES23-198.htm | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-014-6DF44BA78D` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/04/Eplans-Submittal-Requirements-Effective-May-1-2025.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-015-B4713E706E` | https://www.honolulu.gov/dpp/ce/faqs/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-016-B7911C0011` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/10/Customer-Account-Check-Notice.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B47281F6` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-320680/RES23-198,%20CD1.htm | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-018-EED8B53EED` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/How-to-create-ePlans-Account.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC7DA6EB59` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/HOW-TO-SUBMIT-AFTER-APPROVED-TO-PRINT-OR-ATP-IS-GRANTED.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A26DA7040F` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Notice-for-Pre-check-Comments.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AD3E5F8F` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-242934/DOC%20(18).pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-022-4DECF62F89` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/07/Notice-regarding-Permit-Statuses.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-023-2DD9D08DC9` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/08/Permit-Hold-for-Additional-Info.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CCFFDB085` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/12/Special-Assignment-Inspection-Placement-List.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-8306FE8A5A` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/DPP-Third-Party-Residential-Review-Eligibility-Summary.pdf | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20260826T020143Z`:**
- **`SRC-LIVE-CONNECTOR-001-5D6399B840`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"1984 03\" contains seed/workflow terms: building, permit, plan, review, checklist, form, upload."`)
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `")8%0c%3&89;5b-7'e%<+7f7(6h89%:%\"a)8)e%09'*%6-i)73%96-j+'/%5b-7'e%<+7f7(6 d$k&51lm125!%n&o&p&#m5&d$\"!%% 89+:;%<%=>?@a%b3c%ad>efg%h3ij93kl%m33n =>?@a%op%h3ij93k%qr?9i3a%s3kltkl%b3c%<e>?9i3k%ue>r39av%wxyz{|}%~ € ‚ƒ„…†‡ˆ%‰Š‹Œ Ž  ‘’“”•–—˜™š›œ žŸ ¡¢£¤¥¦§¨©ª«¬­®¯°±² ³´µ¶·¸¹º»¼½¾¿ÀÁÂÃÄÅÆÇÈÉÊËÌÍÎÏÐÑÒÓÔÕÖ×ØÙÚÛÜÝÞßàáâãäåæçèéêëìíîïðñòóôõö÷øùúûüýþÿ"`
- **`SRC-LIVE-CONNECTOR-004-9BBFF3FE13`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"dpp permit center appointment system\" contains seed/workflow terms: building, permit, plan, application, appointment, form, queue, upload."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["applicant, submitter, or workflow operator", "regulated organization or operating entity"]`, `artifacts=["application", "form", "plan"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `"Beginning January 2, 2019, the Department of Planning and Permitting (DPP) will require an appointment to apply for a building permit at the Honolulu Permit Center on Fasi Municipal Building, 650 South King Street. Only applicants applying for a commercial building permit with an estimated value of $500,000 or less, or a one- or two-family dwelling building permit that is non-structural in nature, may make an appointment to apply at the Honolulu Permit Center. All other commercial and residential building permit applications will be dropped off in the Permit Center and routed to the appropriate review agencies."`
- **`SRC-LIVE-CONNECTOR-005-221DC0E8FC`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Building Permit Check List Residential 2013\" contains seed/workflow terms: building, permit, checklist, form, upload."`)
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `"Building Permit Check List Residential 2013 (PDF document from https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Check-List-Residential-2013.pdf; binary PDF stream could not be decoded into plain text offline)."`
- **`SRC-LIVE-CONNECTOR-006-78F726C3EA`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Building Permit Format Checklist 2021\" contains seed/workflow terms: building, permit, checklist, format, form, upload."`)
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `"Building Permit Format Checklist 2021 (PDF document from https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Format-Checklist-2021.pdf; binary PDF stream could not be decoded into plain text offline)."`
- **`SRC-LIVE-CONNECTOR-008-1884276FE4`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"prescriptive pv recovery system\" contains seed/workflow terms: building, permit, plan, review, checklist, application, compliance, department, form, manual, packet, submit, upload."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["checklist", "plan", "form", "application", "packet"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"Prescriptive PV Recovery System Packet and Checklist for Department of Planning and Permitting building permit application and plan review.\"."`
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `"Prescriptive PV Recovery System Packet and Checklist for Department of Planning and Permitting building permit application and plan review. Submit completed form, system manual, and compliance checklist with upload package."`
- **`SRC-LIVE-CONNECTOR-013-966C5F8EE1`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"NO.\" contains seed/workflow terms: building, permit, plan, review, application, backlog, code, council, department, file, manual, queue, report, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"WHEREAS, the Department of Planning and Permitting (\"DPP\") is responsible for reviewing applications and plans and issuing building permits for construction projects on Oahu; and WHEREAS, the DPP has experienced a longstanding backlog in processing building permit applications, resulting in significant delays for homeowners and businesses; and WHEREAS, the DPP is seeking to modernize its building permit review process and reduce the permit queue..."`
- **`SRC-LIVE-CONNECTOR-014-6DF44BA78D`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Eplans Submittal Requirements Effective May 1 2025\" contains seed/workflow terms: plan, format, form, submittal, upload."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"regulated organization or operating entity"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["regulated organization or operating entity"]`, `artifacts=["plan", "form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared (was `["official documentation or process exists"]`)
  - **Retained Excerpt:** `"Eplans Submittal Requirements Effective May 1 2025 (PDF document from https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/04/Eplans-Submittal-Requirements-Effective-May-1-2025.pdf; binary PDF stream could not be decoded into plain text offline)."`

---

#### Run 2: `OE-CONTROLLED-LIVE-RESEARCH-20260828T220640Z`
- **Before (`main`):** `{"No Candidate":12,"Reject":3,"Park":10,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":3,"Reject":0,"Park":22,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **12** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-2B70FEBCBF` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Guidelines-To-Complete-Flood-Hazard-Assessment-Report-rev.9.25.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-002-0251C39B5A` | https://www.governmentjobs.com/careers/honolulu/classspecs/763638 | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-003-2DF9C0D6D6` | https://www.governmentjobs.com/careers/honolulu/jobs/2491292-0/building-permit-review-system-assistant | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-004-7ABCE2D07E` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/1984-03.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-005-2EA5FE0882` | https://www.honolulu.gov/dpp/permitting/building-permits/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-006-BB784F301F` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2026/05/dpp_permit_center_appointment_system.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-007-4CD300A8AD` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/Solar-photovoltaic-checklist.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-008-A123029601` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/04/Eplans-Submittal-Requirements-Effective-May-1-2025.pdf | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-009-456F5B78DA` | https://www.honolulu.gov/dpp/ce/faqs/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-010-ECDFB49D4D` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Notice-for-Pre-check-Comments.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-011-B83CE140AC` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-317072/RES23-198.htm | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-012-A59ED22D9F` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-320680/RES23-198,%20CD1.htm | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-013-96B3D712AD` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Check-List-Residential-2013.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-014-6E299E5F47` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Format-Checklist-2021.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-015-BF196CF2B8` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/How-to-create-ePlans-Account.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-016-B76E3A6955` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/HOW-TO-SUBMIT-AFTER-APPROVED-TO-PRINT-OR-ATP-IS-GRANTED.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-017-D4316B3A91` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/10/Customer-Account-Check-Notice.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-018-6348735455` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-242934/DOC%20(18).pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-019-9A4BA25987` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/prescriptive_pv_recovery_system.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-020-A266A538C8` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/07/Notice-regarding-Permit-Statuses.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-021-BBFA386F94` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/08/Permit-Hold-for-Additional-Info.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-022-8269C249F4` | https://www.honolulu.gov/rep/site/dpp/dpp_docs/plan-review-fee-when-switching-form-opt-to-building-permit.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-023-294E616E2F` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-279712/31198.pdf | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-024-8CCFFDB085` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/12/Special-Assignment-Inspection-Placement-List.pdf | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-025-8306FE8A5A` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/DPP-Third-Party-Residential-Review-Eligibility-Summary.pdf | `No Candidate` | **`Park`** |

**Exact triggers for sources newly reaching `Park` in `20260828T220640Z`:**
- **`SRC-LIVE-CONNECTOR-004-7ABCE2D07E`** (`Reject` → **`Park`**): Same PDF as `001-5D6399B840` in Run 1; `officialSystemAbsorptionSignals` cleared.
- **`SRC-LIVE-CONNECTOR-006-BB784F301F`** (`Reject` → **`Park`**): Same appointment-system PDF as `004-9BBFF3FE13` in Run 1; `officialSystemAbsorptionSignals` cleared; `bounded_workflow_identity=proven`.
- **`SRC-LIVE-CONNECTOR-008-A123029601`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Eplans Submittal Requirements Effective May 1 2025\" contains seed/workflow terms: building, permit, plan, review, format, application, apply, code, compliance, department, file, form, manual, resubmit,routing, status, submit, submittal, upload."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["applicant, submitter, or workflow operator", "public agency, county department, or compliance owner"]`, `artifacts=["application", "form", "plan"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"In addition, the ProjectDox software was recently updated to a newer version that changes how plans and documents are uploaded, organized, and reviewed. Complete the Building Permit Application online.\"."`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"In addition, the ProjectDox software was recently updated to a newer version that changes how plans and documents are uploaded, organized, and reviewed. Complete the Building Permit Application online. Upload all construction plans and required documents into the appropriate folders according to the file naming and format standards below. Failure to follow these submittal requirements will result in your application being returned during prescreen before plan review begins."`
- **`SRC-LIVE-CONNECTOR-011-B83CE140AC`** (`No Candidate` → **`Park`**): Same RES23-198 resolution as `013-966C5F8EE1` in Run 1 (`operatorOwner` fallback + `bounded_workflow_identity=proven`).
- **`SRC-LIVE-CONNECTOR-012-A59ED22D9F`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"NO.\" contains seed/workflow terms: building, permit, plan, review, application, backlog, code, council, department, file, manual, queue, report, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"WHEREAS, the Department of Planning and Permitting (\"DPP\") is responsible for reviewing applications and plans and issuing building permits for construction projects on Oahu; and WHEREAS, the DPP has experienced a longstanding backlog in processing building permit applications, resulting in significant delays for homeowners and businesses; and WHEREAS, the Council finds that addressing the building permit backlog requires immediate action..."`
- **`SRC-LIVE-CONNECTOR-015-BF196CF2B8`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"How to create ePlans Account\" contains seed/workflow terms: building, permit, plan, review, application, apply, department, file, form, status, submit, upload."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"To apply for a Building Permit online using the ePlans system, you must first create an ePlans account and submit a building permit application form. Once your application is processed by the Department of Planning and Permitting, you will receive an email invitation to upload your plans and supporting files for review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-017-D4316B3A91`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Customer Account Check Notice\" contains seed/workflow terms: building, permit, plan, review, application, apply, department, form, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Customer Account Check Notice: Before submitting a building permit application for plan review, please verify that your customer account information and email address match the ePlans account registered with the Department of Planning and Permitting. Mismatched account details can delay status notifications and form processing."`
- **`SRC-LIVE-CONNECTOR-018-6348735455`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"DOC (18)\" contains seed/workflow terms: building, permit, plan, review, application, backlog, code, council, department, file, form, manual, report, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"Report to the Honolulu City Council regarding Department of Planning and Permitting building permit application backlog, plan review delays, manual form intake, and status tracking improvements."`
- **`SRC-LIVE-CONNECTOR-019-9A4BA25987`** (`No Candidate` → **`Park`**): Same `prescriptive_pv_recovery_system.pdf` as `008-1884276FE4` in Run 1.
- **`SRC-LIVE-CONNECTOR-020-A266A538C8`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Notice regarding Permit Statuses\" contains seed/workflow terms: building, permit, plan, review, application, department, form, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Notice regarding Permit Statuses: The Department of Planning and Permitting provides online status updates for building permit applications undergoing plan review. Please check the status portal before submitting duplicate inquiry forms."`
- **`SRC-LIVE-CONNECTOR-023-294E616E2F`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"31198\" contains seed/workflow terms: building, permit, plan, review, application, code, council, department, file, form, report, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"Department of Planning and Permitting communication to City Council regarding building permit application review procedures, plan submission requirements, and code compliance reporting."`
- **`SRC-LIVE-CONNECTOR-025-8306FE8A5A`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"DPP Third Party Residential Review Eligibility Summary\" contains seed/workflow terms: building, permit, plan, review, application, code, compliance, department, form, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"DPP Third Party Residential Review Eligibility Summary: Summary of eligibility requirements and compliance forms for submitting residential building permit applications and plans through the Department of Planning and Permitting third-party plan review program."`

---

#### Run 3: `OE-CONTROLLED-LIVE-RESEARCH-20260829T003007Z`
- **Before (`main`):** `{"No Candidate":25,"Reject":0,"Park":0,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":21,"Reject":0,"Park":4,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **4** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-C1B9989060` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/1984-03.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-002-79F65F892D` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Guidelines-To-Complete-Flood-Hazard-Assessment-Report-rev.9.25.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-003-270341CE1E` | https://www.honolulu.gov/dpp/permitting/building-permits/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-004-9BBFF3FE13` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2026/05/dpp_permit_center_appointment_system.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-005-221DC0E8FC` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Check-List-Residential-2013.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-006-78F726C3EA` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/Building-Permit-Format-Checklist-2021.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-007-FB02904BD1` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/Solar-photovoltaic-checklist.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-008-1884276FE4` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/prescriptive_pv_recovery_system.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-009-3D4A582C0E` | https://www.governmentjobs.com/careers/honolulu/jobs/2491292-0/building-permit-review-system-assistant | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-010-FF090425F5` | https://www.governmentjobs.com/careers/honolulu/classspecs/763638 | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-011-7C1D728F41` | https://www.honolulu.gov/rep/site/dpp/dpp_docs/plan-review-fee-when-switching-form-opt-to-building-permit.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E52C66E0A` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-279712/31198.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-966C5F8EE1` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-317072/RES23-198.htm | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-014-6DF44BA78D` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/04/Eplans-Submittal-Requirements-Effective-May-1-2025.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-015-B4713E706E` | https://www.honolulu.gov/dpp/ce/faqs/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-016-B7911C0011` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/10/Customer-Account-Check-Notice.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B47281F6` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-320680/RES23-198,%20CD1.htm | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-018-EED8B53EED` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/How-to-create-ePlans-Account.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC7DA6EB59` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/HOW-TO-SUBMIT-AFTER-APPROVED-TO-PRINT-OR-ATP-IS-GRANTED.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A26DA7040F` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Notice-for-Pre-check-Comments.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AD3E5F8F` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-242934/DOC%20(18).pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-022-4DECF62F89` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/07/Notice-regarding-Permit-Statuses.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-023-2DD9D08DC9` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/08/Permit-Hold-for-Additional-Info.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CCFFDB085` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/12/Special-Assignment-Inspection-Placement-List.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-8306FE8A5A` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/DPP-Third-Party-Residential-Review-Eligibility-Summary.pdf | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20260829T003007Z`:**
- **`SRC-LIVE-CONNECTOR-009-3D4A582C0E`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=job_task_signal`, `supportsClaim="\"Job Bulletin\" contains seed/workflow terms: application, apply, building, code, compliance, department, file, form, manual, permit, plan, review, routing, status, submit."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["building permit counter / review support staff", "public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"Assists in the pre-screening, intake, routing, and tracking of building permit applications and construction plans submitted to the Department of Planning and Permitting; checks application forms and supporting documents for completeness and compliance with submittal requirements..."`
- **`SRC-LIVE-CONNECTOR-010-FF090425F5`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=job_task_signal`, `supportsClaim="\"Class Specifications\" contains seed/workflow terms: application, building, code, compliance, department, file, form, manual, permit, plan, report, review, routing, status."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["building permit counter / review support staff", "public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"Reviews building permit applications, plans, and specifications for completeness; compiles and maintains permit records and files; coordinates and tracks permits routed to various agencies for review and approval..."`
- **`SRC-LIVE-CONNECTOR-013-966C5F8EE1`** & **`SRC-LIVE-CONNECTOR-017-D5B47281F6`** (`No Candidate` → **`Park`**): Same Honolulu City Council RES23-198 & RES23-198 CD1 resolutions as in Runs 1 & 2 (`operatorOwner` fallback + `bounded_workflow_identity=proven`).

---

#### Run 4: `OE-CONTROLLED-LIVE-RESEARCH-20261014T000148Z`
- **Before (`main`):** `{"No Candidate":23,"Reject":1,"Park":1,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":17,"Reject":0,"Park":8,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **7** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-C800B41CD4` | https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-7D322108EB` | https://www.hud.gov/hud-partners/community-programs | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-003-21B9FE9F98` | https://www.acquisition.gov/far-overhaul/far-part-deviation-guide | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-004-7759B5D2FA` | https://www.acquisition.gov/content/part-19-small-business-programs | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-005-25614C7823` | https://www.ecfr.gov/current/title-49/subtitle-A/part-26 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-006-7F24938607` | https://www.ecfr.gov/current/title-24/subtitle-A/part-58 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-007-FF5FBFDBB2` | https://www.cms.gov/medicare/coding-billing/hipaa-administrative-simplification | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-008-19E1A33A8E` | https://www.cms.gov/medicare/payment/fee-schedules/physician | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-009-396BD8CA24` | https://www.transportation.gov/civil-rights/disadvantaged-business-enterprise/dbe-guidance | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-010-FF090425F5` | https://www.governmentjobs.com/careers/honolulu/classspecs/763638 | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-011-76C1FB0970` | https://www.fema.gov/grants/mitigation/learn/hazard-mitigation-assistance-guidance | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E5C45F155` | https://www.fema.gov/emergency-managers/national-preparedness/exercises | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-965743FE9E` | https://www.epa.gov/npdes/npdes-stormwater-program | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-014-6DFEFAD618` | https://www.epa.gov/hwgenerators/hazardous-waste-manifest-system | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-015-B47F68FEAC` | https://www.osha.gov/recordkeeping | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-016-B79CD5F8AC` | https://www.osha.gov/injuryreporting | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B4AC1D37` | https://www.sam.gov/content/entity-registration | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-018-EED8B53EED` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/11/How-to-create-ePlans-Account.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC7DA6EB59` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/08/HOW-TO-SUBMIT-AFTER-APPROVED-TO-PRINT-OR-ATP-IS-GRANTED.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A26DA7040F` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/09/Notice-for-Pre-check-Comments.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AD3E5F8F` | https://www4.honolulu.gov/docushare/dsweb/Get/Document-242934/DOC%20(18).pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-022-4DECF62F89` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/07/Notice-regarding-Permit-Statuses.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-023-2DD9D08DC9` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2025/08/Permit-Hold-for-Additional-Info.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CCFFDB085` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2023/12/Special-Assignment-Inspection-Placement-List.pdf | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-8306FE8A5A` | https://www.honolulu.gov/dpp/wp-content/uploads/sites/67/2024/11/DPP-Third-Party-Residential-Review-Eligibility-Summary.pdf | `No Candidate` | **`Park`** |

**Exact triggers for sources newly reaching `Park` in `20261014T000148Z`:**
- **`SRC-LIVE-CONNECTOR-001-C800B41CD4`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"CISA Tabletop Exercise Packages | Cybersecurity and Infrastructure Security Agency CISA\" contains seed/workflow terms: agency, compliance, department, file, form, manual, package, plan, report, review, template."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "package"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"CISA Tabletop Exercise Packages (CTEPs) are a comprehensive set of resources designed to assist stakeholders in conducting their own tabletop exercises. Each package includes adaptable templates, situation manuals, facilitator guides, and after-action report forms.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-002-7D322108EB`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Community Planning and Development Programs | HUD.gov / U.S. Department of Housing and Urban Development (HUD)\" contains seed/workflow terms: agency, application, apply, compliance, department, form, grant, plan, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Community Planning and Development (CPD) administers grant programs that help communities plan and finance housing, community development, and homelessness assistance. Grantees submit consolidated plans, annual action plans, and compliance reporting forms to HUD for review."`
- **`SRC-LIVE-CONNECTOR-003-21B9FE9F98`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"FAR Part Deviation Guide | Acquisition.GOV\" contains seed/workflow terms: agency, code, compliance, contract, department, file, form, manual, procurement, report, review, routing, submit."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["applicant, submitter, or workflow operator", "public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"FAR Part Deviation Guide for federal agency contracting officers and procurement teams submitting class or individual deviation requests and compliance forms for review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-007-FF5FBFDBB2`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"HIPAA Administrative Simplification | CMS\" contains seed/workflow terms: code, compliance, department, electronic, file, form, manual, plan, provider, report, review, standard, status, submit, transaction."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"HIPAA Administrative Simplification establishes national standards for electronic health care transactions, code sets, unique identifiers, and operating rules. Health plans, clearinghouses, and health care providers use these standards to submit claims, check eligibility status, and file compliance reports."`
- **`SRC-LIVE-CONNECTOR-009-396BD8CA24`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"DBE Guidance | US Department of Transportation\" contains seed/workflow terms: agency, application, certification, code, compliance, contract, department, file, form, manual, plan, procurement, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Official DOT Disadvantaged Business Enterprise (DBE) program guidance, certification application forms, contract goal reporting requirements, and compliance review procedures for state and local transportation recipients.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-013-965743FE9E`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"NPDES Stormwater Program | US EPA\" contains seed/workflow terms: agency, application, apply, code, compliance, construction, department, electronic, file, form, manual, permit, plan, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"The NPDES Stormwater Program regulates stormwater discharges from municipal separate storm sewer systems (MS4s), construction activities, and industrial activities. Operators must develop a Stormwater Pollution Prevention Plan (SWPPP) and submit Notice of Intent (NOI) permit application forms and compliance reports."`
- **`SRC-LIVE-CONNECTOR-025-8306FE8A5A`** (`No Candidate` → **`Park`**): Same `DPP-Third-Party-Residential-Review-Eligibility-Summary.pdf` as in Run 2.

---

#### Run 5: `OE-CONTROLLED-LIVE-RESEARCH-20261014T203425Z`
- **Before (`main`):** `{"No Candidate":23,"Reject":2,"Park":0,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":13,"Reject":0,"Park":12,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **12** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-C800B41CD4` | https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-7D322108EB` | https://www.hud.gov/hud-partners/community-programs | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-003-21B9FE9F98` | https://www.acquisition.gov/far-overhaul/far-part-deviation-guide | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-004-985F19DF6B` | https://www.gsa.gov/buy-through-us/purchasing-programs/multiple-award-schedule | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-005-2A7840EB95` | https://www.sba.gov/federal-contracting/contracting-assistance-programs/8a-business-development-program | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-006-74CD1A5E8A` | https://www.dol.gov/agencies/whd/government-contracts/construction | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-007-FF5FBFDBB2` | https://www.cms.gov/medicare/coding-billing/hipaa-administrative-simplification | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-008-19E1A33A8E` | https://www.cms.gov/medicare/payment/fee-schedules/physician | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-009-396BD8CA24` | https://www.transportation.gov/civil-rights/disadvantaged-business-enterprise/dbe-guidance | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-010-FF0C23E993` | https://www.faa.gov/airports/engineering/construction_standards | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-011-76C1FB0970` | https://www.fema.gov/grants/mitigation/learn/hazard-mitigation-assistance-guidance | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E5C45F155` | https://www.fema.gov/emergency-managers/national-preparedness/exercises | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-965743FE9E` | https://www.epa.gov/npdes/npdes-stormwater-program | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-014-6DFEFAD618` | https://www.epa.gov/hwgenerators/hazardous-waste-manifest-system | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-015-B47F68FEAC` | https://www.osha.gov/recordkeeping | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-016-B79CD5F8AC` | https://www.osha.gov/injuryreporting | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B4AC1D37` | https://www.sam.gov/content/entity-registration | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-018-EEA2B289F4` | https://www.usaspending.gov/search | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC782D32A7` | https://www.grants.gov/learn-grants/grant-making-agencies.html | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A2648E2F49` | https://www.regulations.gov/faq | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AEF7D43C` | https://www.foia.gov/how-to.html | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-022-4DE1A293C4` | https://www.cio.gov/policies-and-priorities/fedramp/ | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-023-2DD8340F1C` | https://www.nist.gov/cyberframework | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CC1E43B29` | https://www.healthit.gov/topic/certification-ehrs/certification-health-it | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-830F921E4D` | https://www.fcc.gov/licensing-databases/forms | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20261014T203425Z`:**
- **`SRC-LIVE-CONNECTOR-001-C800B41CD4`, `002-7D322108EB`, `003-21B9FE9F98`, `007-FF5FBFDBB2`, `009-396BD8CA24`, `013-965743FE9E`**: Same triggers and excerpts as Run 4 (`20261014T000148Z`).
- **`SRC-LIVE-CONNECTOR-004-985F19DF6B`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Multiple Award Schedule | GSA\" contains seed/workflow terms: agency, application, apply, compliance, contract, department, file, form, manual, package, plan, procurement, report, review, schedule, status, submit, template, upload, vendor."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application", "package"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Under the GSA Multiple Award Schedule (MAS) program, GSA issues long-term governmentwide contracts that provide federal, state, and local government buyers access to commercial products, services, and solutions at pre-negotiated pricing. Vendors must prepare and submit an offer package including technical, pricing, and compliance forms.\"."`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"Under the GSA Multiple Award Schedule (MAS) program, GSA issues long-term governmentwide contracts that provide federal, state, and local government buyers access to commercial products, services, and solutions at pre-negotiated pricing. Vendors must prepare and submit an offer package including technical, pricing, and compliance forms.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-005-2A7840EB95`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"8(a) Business Development program | U.S. Small Business Administration\" contains seed/workflow terms: agency, application, apply, certification, compliance, contract, department, file, form, plan, procurement, report, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"To apply for the 8(a) Business Development program, small businesses must submit an electronic application, business plan, and supporting financial and compliance forms for SBA review and annual continuing eligibility reporting.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-006-74CD1A5E8A`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Davis-Bacon and Related Acts | U.S. Department of Labor\" contains seed/workflow terms: agency, code, compliance, construction, contract, contractor, department, file, form, manual, payroll, report, reporting, review, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"The Davis-Bacon and Related Acts apply to contractors and subcontractors performing on federally funded or assisted contracts in excess of $2,000 for the construction, alteration, or repair of public buildings or public works. Contractors must submit weekly certified payroll reports (Form WH-347) to the contracting agency.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-010-FF0C23E993`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Airport Construction Standards | Federal Aviation Administration\" contains seed/workflow terms: agency, application, code, compliance, construction, contract, contractor, department, file, form, manual, plan, report, review, specification, standard, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"FAA Advisory Circulars and construction standards for airport development projects, contractor quality control plans, specification compliance forms, and engineering review submittals.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-021-E6AEF7D43C`** (`Reject` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"How to Make a FOIA Request | FOIA.gov\" contains seed/workflow terms: agency, application,department, file, form, portal, fee, report, request, review, status, submit."`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["applicant, submitter, or workflow operator", "public agency, county department, or compliance owner"]`, `artifacts=["form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Submit a Freedom of Information Act (FOIA) request to a federal agency by describing the records you seek, specifying fee waiver or fee category information, and submitting the agency request form for review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-022-4DE1A293C4`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"FedRAMP | CIO.GOV\" contains seed/workflow terms: agency, authorization, cloud, compliance, department, file, form, manual, package, plan, provider, report, review, security, standard, status, submit, template."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "package"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"The Federal Risk and Authorization Management Program (FedRAMP) provides a standardized approach to security assessment, authorization, and continuous monitoring for cloud products and services. Cloud Service Providers (CSPs) must compile a System Security Plan (SSP) authorization package and monthly continuous monitoring reports for agency review.\"."`
  - `officialSystemAbsorptionSignals` cleared

---

#### Run 6: `OE-CONTROLLED-LIVE-RESEARCH-20261014T203950Z`
- **Before (`main`):** `{"No Candidate":17,"Reject":8,"Park":0,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":12,"Reject":0,"Park":13,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **13** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-323C7947F5` | https://www.acquisition.gov/far-overhaul/far-part-deviation-guide | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-7D322108EB` | https://www.hud.gov/hud-partners/community-programs | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-003-211A3D6E7B` | https://www.cisa.gov/resources-tools/services/cisa-tabletop-exercise-packages | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-004-985F19DF6B` | https://www.gsa.gov/buy-through-us/purchasing-programs/multiple-award-schedule | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-005-2A7840EB95` | https://www.sba.gov/federal-contracting/contracting-assistance-programs/8a-business-development-program | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-006-74CD1A5E8A` | https://www.dol.gov/agencies/whd/government-contracts/construction | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-007-FF5FBFDBB2` | https://www.cms.gov/medicare/coding-billing/hipaa-administrative-simplification | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-008-19E1A33A8E` | https://www.cms.gov/medicare/payment/fee-schedules/physician | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-009-396BD8CA24` | https://www.transportation.gov/civil-rights/disadvantaged-business-enterprise/dbe-guidance | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-010-FF0C23E993` | https://www.faa.gov/airports/engineering/construction_standards | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-011-76C1FB0970` | https://www.fema.gov/grants/mitigation/learn/hazard-mitigation-assistance-guidance | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-012-7E5C45F155` | https://www.fema.gov/emergency-managers/national-preparedness/exercises | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-965743FE9E` | https://www.epa.gov/npdes/npdes-stormwater-program | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-014-6DFEFAD618` | https://www.epa.gov/hwgenerators/hazardous-waste-manifest-system | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-015-B47F68FEAC` | https://www.osha.gov/recordkeeping | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-016-B79CD5F8AC` | https://www.osha.gov/injuryreporting | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B4AC1D37` | https://www.sam.gov/content/entity-registration | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-018-EEA2B289F4` | https://www.usaspending.gov/search | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC782D32A7` | https://www.grants.gov/learn-grants/grant-making-agencies.html | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A2648E2F49` | https://www.regulations.gov/faq | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AEF7D43C` | https://www.foia.gov/how-to.html | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-022-4DE1A293C4` | https://www.cio.gov/policies-and-priorities/fedramp/ | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-023-2DD8340F1C` | https://www.nist.gov/cyberframework | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CC1E43B29` | https://www.healthit.gov/topic/certification-ehrs/certification-health-it | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-830F921E4D` | https://www.fcc.gov/licensing-databases/forms | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20261014T203950Z`:**
- **`001-323C7947F5`, `002-7D322108EB`, `003-211A3D6E7B`, `004-985F19DF6B`, `005-2A7840EB95`, `006-74CD1A5E8A`, `007-FF5FBFDBB2`, `009-396BD8CA24`, `010-FF0C23E993`, `013-965743FE9E`, `021-E6AEF7D43C`, `022-4DE1A293C4`**: Same URLs, triggers, and excerpts as Run 5 (`20261014T203425Z`).
- **`SRC-LIVE-CONNECTOR-011-76C1FB0970`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Hazard Mitigation Assistance Guidance | FEMA.gov\" contains seed/workflow terms: agency, application, apply, code, compliance, department, file, form, grant, manual, plan, project, report, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"FEMA Hazard Mitigation Assistance (HMA) Program and Policy Guide provides comprehensive eligibility, application submittal, benefit-cost analysis, environmental planning and historic preservation compliance, and quarterly reporting requirements for state, local, tribal, and territorial subapplicants."`

---

#### Run 7: `OE-CONTROLLED-LIVE-RESEARCH-20261019T144637Z`
- **Before (`main`):** `{"No Candidate":20,"Reject":2,"Park":3,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":8,"Reject":0,"Park":17,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **14** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-F733BD9AC7` | https://www.sba.gov/article/2026/mar/05/sba-suspends-more-1000-contractors-187m-8a-program-Quick | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-01C2BD3EB1` | https://www.sba.gov/federal-contracting/contracting-guide/assess-your-business | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-003-8E050E2F9A` | https://www.sba.gov/federal-contracting/contracting-assistance-programs/sba-mentor-protege-program | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-004-6E33F2670F` | https://www.sba.gov/federal-contracting/contracting-assistance-programs/small-disadvantaged-business | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-005-D89FE94193` | https://www.sba.gov/about-sba/SBA-newsroom/press-releases-media-advisories/sba-orders-all-8a-participants-provide-financial-records | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-006-70C5D21B73` | https://www.dol.gov/agencies/eta/advisories/tegl-10-23 | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-007-4B8393860E` | https://www.dol.gov/agencies/whd/data/charts/construction | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-008-1988101C3F` | https://www.dol.gov/agencies/whd/government-contracts/prevailing-wage-resource-book/dbra-compliance-principles | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-009-396BD8CA24` | https://www.transportation.gov/civil-rights/disadvantaged-business-enterprise/dbe-guidance | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-010-FF0C23E993` | https://www.faa.gov/airports/engineering/construction_standards | `Reject` | **`Park`** |
| `SRC-LIVE-CONNECTOR-011-76C1FB0970` | https://www.fema.gov/grants/mitigation/learn/hazard-mitigation-assistance-guidance | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-012-7E5C45F155` | https://www.fema.gov/emergency-managers/national-preparedness/exercises | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-965743FE9E` | https://www.epa.gov/npdes/npdes-stormwater-program | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-014-6DFEFAD618` | https://www.epa.gov/hwgenerators/hazardous-waste-manifest-system | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-015-B47F68FEAC` | https://www.osha.gov/recordkeeping | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-016-B79CD5F8AC` | https://www.osha.gov/injuryreporting | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-017-D5B4AC1D37` | https://www.sam.gov/content/entity-registration | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-018-EEA2B289F4` | https://www.usaspending.gov/search | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-019-EC782D32A7` | https://www.grants.gov/learn-grants/grant-making-agencies.html | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-020-A2648E2F49` | https://www.regulations.gov/faq | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AEF7D43C` | https://www.foia.gov/how-to.html | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-022-4DE1A293C4` | https://www.cio.gov/policies-and-priorities/fedramp/ | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-023-2DD8340F1C` | https://www.nist.gov/cyberframework | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CC1E43B29` | https://www.healthit.gov/topic/certification-ehrs/certification-health-it | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-025-830F921E4D` | https://www.fcc.gov/licensing-databases/forms | `No Candidate` | **`Park`** |

**Exact triggers for sources newly reaching `Park` in `20261019T144637Z`:**
- **`SRC-LIVE-CONNECTOR-001-F733BD9AC7`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"SBA Suspends More Than 1,000 Contractors in $18.7B 8(a) Program | U.S. Small Business Administration\" contains seed/workflow terms: agency, application, audit, certification, compliance, contract, contractor, department, file, form, manual, portal, procurement, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"The U.S. Small Business Administration announced the suspension of more than 1,000 contractors in the 8(a) Business Development program after participants failed to submit mandatory financial records and annual compliance certification forms through the SBA portal for audit review.\"."`
- **`SRC-LIVE-CONNECTOR-005-D89FE94193`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"SBA Orders All 8(a) Participants to Provide Financial Records | U.S. Small Business Administration\" contains seed/workflow terms: agency, audit, certification, compliance, contract, contractor, department, document, file, form, portal, procurement, record, report, review, submit, upload."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"SBA directed all 8(a) contracting participants to upload three years of financial statements, tax forms, and subcontracting compliance records for mandatory review.\"."`
- **`SRC-LIVE-CONNECTOR-006-70C5D21B73`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"TEGL 10-23 | U.S. Department of Labor\" contains seed/workflow terms: agency, application, compliance, department, file, form, grant, manual, plan, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Training and Employment Guidance Letter (TEGL) 10-23 provides instructions to state workforce agencies for submitting grant application packages, state plans, and quarterly financial and performance compliance reporting forms to the Department of Labor."`
- **`SRC-LIVE-CONNECTOR-008-1988101C3F`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"DBRA Compliance Principles | U.S. Department of Labor\" contains seed/workflow terms: agency, code, compliance, construction, contract, contractor, department, file, form, manual, payroll, report, reporting, review, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Davis-Bacon and Related Acts (DBRA) compliance principles for federal construction contracts, wage determinations, and weekly certified payroll form (WH-347) submission and agency review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-009-396BD8CA24`, `010-FF0C23E993`, `011-76C1FB0970`, `021-E6AEF7D43C`, `022-4DE1A293C4`**: Same URLs, triggers, and excerpts as Run 6 (`20261014T203950Z`).
- **`SRC-LIVE-CONNECTOR-016-B79CD5F8AC`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Report a Fatality or Severe Injury | Occupational Safety and Health Administration\" contains seed/workflow terms: agency, compliance, department, electronic, file, form, manual, online, report, reporting, review, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Employers must report any worker fatality within 8 hours and any amputation, loss of an eye, or hospitalization of a worker within 24 hours to OSHA by telephone or by submitting the serious event reporting form online."`
- **`SRC-LIVE-CONNECTOR-017-D5B4AC1D37`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"SAM.gov | Entity Registration\" contains seed/workflow terms: agency, application, apply, certification, compliance, contract, department, entity, file, form, grant, manual, procurement, registration, report, review, status, submit, validation, vendor."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form", "application"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Register your entity in SAM.gov to bid on federal procurement contracts and apply for federal assistance grants. Entities must complete core data, assertions, representations and certifications forms, and submit entity validation documentation for review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-018-EEA2B289F4`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"USAspending.gov | Award Search\" contains seed/workflow terms: agency, award, compliance, contract, contractor, department, file, form, grant, procurement, recipient, report, reporting, review, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Search federal contract, grant, loan, and other financial assistance award data reported by federal agencies and prime recipients under DATA Act compliance reporting forms.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-019-EC782D32A7`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Grant-Making Agencies | Grants.gov\" contains seed/workflow terms: agency, application, apply, compliance, department, file, form, grant, package, plan, report, review, status, submit, workspace."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["plan", "form", "application", "package"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[package_reality]=proven` (newly drives `proofObject.packageReality="proven"`): `"Requires structured package/checklist output: \"Federal grant-making agencies post funding opportunities on Grants.gov where applicants assemble, validate, and submit standard SF-424 grant application form packages for agency review.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-025-830F921E4D`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"FCC Forms | Federal Communications Commission\" contains seed/workflow terms: agency, application, compliance, department, electronic, fee, file, filing, form, license, manual, report, review, schedule, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form", "application", "schedule"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"Directory of FCC licensing, tariff, fee schedule, and regulatory compliance forms required for electronic filing and commission review.\"."`
  - `officialSystemAbsorptionSignals` cleared

---

#### Run 8: `OE-CONTROLLED-LIVE-RESEARCH-20261021T134618Z`
- **Before (`main`):** `{"No Candidate":17,"Reject":0,"Park":7,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":9,"Reject":0,"Park":15,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **8** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-86228F9FD4` | https://www.acquisition.gov/content/part-4-administrative-and-information-matters | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-002-7D322108EB` | https://www.hud.gov/hud-partners/community-programs | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-003-21B9FE9F98` | https://www.acquisition.gov/far-overhaul/far-part-deviation-guide | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-004-985F19DF6B` | https://www.gsa.gov/buy-through-us/purchasing-programs/multiple-award-schedule | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-005-2A7840EB95` | https://www.sba.gov/federal-contracting/contracting-assistance-programs/8a-business-development-program | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-006-74CD1A5E8A` | https://www.dol.gov/agencies/whd/government-contracts/construction | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-007-FF5FBFDBB2` | https://www.cms.gov/medicare/coding-billing/hipaa-administrative-simplification | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-008-1988101C3F` | https://www.dol.gov/agencies/whd/government-contracts/prevailing-wage-resource-book/dbra-compliance-principles | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-009-396BD8CA24` | https://www.transportation.gov/civil-rights/disadvantaged-business-enterprise/dbe-guidance | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-010-FF0C23E993` | https://www.faa.gov/airports/engineering/construction_standards | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-011-76C1FB0970` | https://www.fema.gov/grants/mitigation/learn/hazard-mitigation-assistance-guidance | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E5C45F155` | https://www.fema.gov/emergency-managers/national-preparedness/exercises | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-965743FE9E` | https://www.epa.gov/npdes/npdes-stormwater-program | `Park` | `Park` |
| `SRC-LIVE-CONNECTOR-014-6DFEFAD618` | https://www.epa.gov/hwgenerators/hazardous-waste-manifest-system | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-015-B47F68FEAC` | https://www.osha.gov/recordkeeping | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-016-B79CD5F8AC` | https://www.osha.gov/injuryreporting | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-017-D5B4AC1D37` | https://www.sam.gov/content/entity-registration | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-018-EEA2B289F4` | https://www.usaspending.gov/search | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-019-EC782D32A7` | https://www.grants.gov/learn-grants/grant-making-agencies.html | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A2648E2F49` | https://www.regulations.gov/faq | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-E6AEF7D43C` | https://www.foia.gov/how-to.html | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-022-4DE1A293C4` | https://www.cio.gov/policies-and-priorities/fedramp/ | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-023-2DD8340F1C` | https://www.nist.gov/cyberframework | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CC1E43B29` | https://www.healthit.gov/topic/certification-ehrs/certification-health-it | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20261021T134618Z`:**
- **`SRC-LIVE-CONNECTOR-001-86228F9FD4`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Part 4 - Administrative and Information Matters | Acquisition.GOV\" contains seed/workflow terms: agency, certification, code, compliance, contract, contractor, department, electronic, file, form, manual, procurement, record, registration, report, reporting, review, status, submit, system, vendor."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["building permitting / construction compliance"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `claim[buyer_evidence]=proven` (newly drives `proofObject.buyerPackageProof="proven"`): `"Contains procurement or fee signal: \"FAR Part 4 prescribes policies and procedures relating to the administrative aspects of contract execution, contractor basic safeguarding of covered contractor information systems, SAM registration, and contract reporting forms.\"."`
  - `officialSystemAbsorptionSignals` cleared
- **`SRC-LIVE-CONNECTOR-008-1988101C3F`, `016-B79CD5F8AC`, `017-D5B4AC1D37`, `018-EEA2B289F4`, `022-4DE1A293C4`**: Same URLs, triggers, and excerpts as Run 7 (`20261019T144637Z`).
- **`SRC-LIVE-CONNECTOR-014-6DFEFAD618`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Hazardous Waste Manifest System | US EPA\" contains seed/workflow terms: agency, compliance, department, electronic, file, form, manual, paper, record, report, review, status, submit, system, tracking."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"EPA’s Hazardous Waste Manifest System tracks hazardous waste from the time it leaves the generator facility where it was produced until it reaches the off-site waste management facility. Generators and receiving facilities submit manifest forms (EPA Form 8700-22) via e-Manifest or paper upload for compliance tracking."`
- **`SRC-LIVE-CONNECTOR-015-B47F68FEAC`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=official_documentation`, `supportsClaim="\"Recordkeeping - Overview | Occupational Safety and Health Administration\" contains seed/workflow terms: agency, compliance, department, electronic, file, form, log, manual, record, report, reporting, review, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["public agency, county department, or compliance owner"]`, `artifacts=["form"]`, `handoff="operator-prepared artifact to official submission or review"`
  - `officialSystemAbsorptionSignals` cleared
  - **Retained Excerpt:** `"Under OSHA's Recordkeeping regulation, covered employers are required to prepare and maintain records of serious occupational injuries and illnesses using the OSHA 300 Log, 300A Summary, and 301 Incident Report forms, and electronically submit annual summary data via the Injury Tracking Application (ITA)."`

---

#### Run 9: `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z`
- **Before (`main`):** `{"No Candidate":24,"Reject":0,"Park":0,"Advance":0}`
- **After (`exp/connector-intake-refinements`):** `{"No Candidate":20,"Reject":0,"Park":4,"Advance":0}`
- **Newly reaching `Park` or `Advance`:** **4** (`Park`)

| Source ID | Source URL | Decision Before (`main`) | Decision After (Branch) |
| --- | --- | --- | --- |
| `SRC-LIVE-CONNECTOR-001-F7B6BA7E53` | https://www.biggerpockets.com/forums/52/topics/836888-process-around-property-management-maintenance-calls | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-002-7CA088C276` | https://quickbooks.intuit.com/learn-support/en-us/other-questions/recurring-billing-which-requires-a-manual-edit-for-hourly-usage/00/503179 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-003-2353257828` | https://community.shopify.com/c/shopify-discussions/Help-us-solve-pain-points-for-shopify-store-owners/td-p/507257 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-004-9A0A107428` | https://www.thetruckersreport.com/truckingindustryforum/threads/issues-between-driver-and-dispatcher.260260/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-005-52C61A1B46` | https://www.aapc.com/codes/coding-newsletters/my-cardiology-coding-alert/claims-sidestep-duplicate-claim-denials-by-properly-appending-repeat-modifiers-172134-article | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-006-789EC40A59` | https://www.biggerpockets.com/forums/850/topics/895663-handling-maintenance-coordinate-directly-or-get-the-middle | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-007-FB0C7A82F9` | https://www.biggerpockets.com/forums/52/topics/1156431-The-Landlord-s-Guide-to-Effective-Maintenance-Triage | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-008-1834517A75` | https://quickbooks.intuit.com/learn-support/en-us/other-questions/recurring-invoices-for-next-month-for-all-monthly-fees-plus-this/00/188427 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-009-3D4C844446` | https://community.shopify.com/c/accounting-and-taxes/best-process-to-sync-quot-sales-quot-and-quot-cogs-quot-data/td-p/1357713 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-010-8E3608361C` | https://community.shopify.com/c/accounting-and-taxes/calculating-fulfilled-orders-by-date-for-accounting-purposes/td-p/1508153 | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-011-7C1E748FB6` | https://www.biggerpockets.com/forums/52/topics/785719-landlords-who-self-manage-handling-maintenance-requests | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-012-7E52C66E0A` | https://www.thetruckersreport.com/truckingindustryforum/threads/Do-Any-Companies-Still-Use-Manual-Logs-Paper-Logs.2467831/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-013-966C5F8EE1` | https://community.shopify.com/c/shopify-discussions/ inventory-syncing-problem/m-p/2308432 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-014-6DF44BA78D` | https://www.thetruckersreport.com/truckingindustryforum/threads/broker-load-board-pain-points.2491234/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-015-2FF8140F22` | https://www.aapc.com/codes/coding-newsletters/my-general-surgery-coding-alert/compliance-fix-your-duplicate-claim-denial-problem-with-modifiers-172567-article | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-016-B7911C0011` | https://quickbooks.intuit.com/learn-support/en-us/bank-feeds/matching-payouts-with-multiple-invoices-and-fees/00/1129483 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-017-D5B47281F6` | https://www.biggerpockets.com/forums/52/topics/1092341-tenant-maintenance-portal-vs-texting | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-018-EED8B53EED` | https://www.thetruckersreport.com/truckingindustryforum/threads/detention-pay-tracking-and-bol-uploads.2510922/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-019-EC7DA6EB59` | https://www.thetruckersreport.com/truckingindustryforum/threads/ifta-fuel-tax-reporting-spreadsheet-or-software.2489102/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-020-A26DA7040F` | https://www.aapc.com/discuss/threads/prior-authorization-tracking-spreadsheet.192837/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-021-A783238AED` | https://www.aapc.com/codes/coding-newsletters/my-general-surgery-coding-alert/compliance-follow-this-advice-for-clean-medicare-claims-173609-article | `No Candidate` | **`Park`** |
| `SRC-LIVE-CONNECTOR-022-4DECF62F89` | https://community.shopify.com/c/accounting-and-taxes/reconciling-shopify-payouts-in-quickbooks-online-with-fees-and/td-p/1892341 | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-023-2DD9D08DC9` | https://www.aapc.com/discuss/threads/denial-management-workflow-for-small-billing-company.198412/ | `No Candidate` | `No Candidate` |
| `SRC-LIVE-CONNECTOR-024-8CCFFDB085` | https://quickbooks.intuit.com/learn-support/en-us/reports-and-accounting/exporting-custom-reports-to-excel-and-reformatting-every-week/00/1203948 | `No Candidate` | `No Candidate` |

**Exact triggers for sources newly reaching `Park` in `20261028T222943Z`:**
- **`SRC-LIVE-CONNECTOR-005-52C61A1B46`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=operator_pain`, `supportsClaim="\"Sidestep Duplicate Claim Denials by Properly Appending Repeat Modifiers : Claims\" contains seed/workflow terms: apply, code, contract, form, manual, report, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`, triggered by `"department"` and `"provider"`)
  - **Retained Excerpt:** `"If you’ve ever had issues with submitting duplicate claims in your practice, Michelle Coleman, CPC, and Arlene Dunphy, CPC, provider outreach and education consultants from the Medicare Administrative Contractor (MAC) National Government Services (NGS), recently shared some helpful advice in the webinar “How to Avoid Duplicate Claim Denials.” . “We get a report once a month of the top 100 providers who have submitted the most duplicate claims,” according to Coleman. “We review the report, and if you are on that report, you could be getting a call from the provider outreach and education department."`
- **`SRC-LIVE-CONNECTOR-010-8E3608361C`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=public_discussion`, `supportsClaim="\"Calculating fulfilled orders by date for accounting purposes\" contains seed/workflow terms: workflow, form, report, reporting, status, vendor."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"regulated organization or operating entity"` (from `likelyBuyer`, triggered by `"vendor"`)
  - **Retained Excerpt:** `"Current tool: Report Pundit “Fulfilled orders” report includes fulfilled date but appears inaccurate. OP is open to revising workflow (e.g., cancel/recreate orders when changes occur) to improve reporting accuracy. Suggestion: Use the Report Toaster app to produce final sales totals for orders fulfilled within a period; vendor offers setup assistance via live chat."`
- **`SRC-LIVE-CONNECTOR-015-2FF8140F22`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=operator_pain`, `supportsClaim="\"Fix Your Duplicate Claim Denial Problem with Modifiers : Compliance\" contains seed/workflow terms: code, compliance, contract, form, manual, report, status, submit."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"public agency, county department, or compliance owner"` (from `likelyBuyer`, triggered by `"department"` and `"provider"`)
  - **Retained Excerpt:** `"“We get a report once a month of the top 100 providers who have submitted the most duplicate claims,” according to Coleman. “We review the report, and if you are on that report, you could be getting a call from the provider outreach and education department. However, Coleman added that if they see you are still submitting duplicate claims after the provider outreach department has spoken to you, you could be identified as an abusive biller and be investigated for fraud."`
- **`SRC-LIVE-CONNECTOR-021-A783238AED`** (`No Candidate` → **`Park`**):
  - `workflowReality=proven` (`rawSignal.confidence >= 0.5`; `category=operator_pain`, `supportsClaim="\"Follow This Advice for Clean Medicare Claims : Compliance\" contains seed/workflow terms: application, code, compliance, contract, file, form, replacement, report."`)
  - `operatorOwner` fallback: `"unknown operator role; needs direct validation"` → `"regulated organization or operating entity"` (from `likelyBuyer`, triggered by `"provider"`)
  - `claim[bounded_workflow_identity]=proven`: `industries=["SaaS workflow evidence"]`, `actors=["regulated organization or operating entity"]`, `artifacts=["form"]` (matched `"CMS-1500 claim form"`), `handoff="operator-prepared artifact to official submission or review"`
  - **Retained Excerpt:** `"You’ll have to track down that information and submit the claim accordingly. CO-22: Check the eligibility file for information on the patient’s primary insurance (if still working) and after settling with the primary, submit a secondary claim to Medicare. On the other hand, if the insured is retired, submit a Medicare primary claim and report the retirement date in Item 11b of the CMS-1500 claim form or the electronic equivalent."`

---

### 3. Why `operatorRoleGuess` Was `"unknown"` for All 24 Sources in `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z`, and How to Fix Non-Government Role Inference

#### Root Cause
In [AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L90-L111) and `inferOperatorRole(text, matchedRoleTerms)` ([AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L1748-L1767)), operator role inference is hardcoded to **12 government/permitting terms** across four checks:
1. **Honolulu DPP building-permit job-bulletin phrases:** `"permit technician"`, `"permit counter"`, `"files and maintains"`, `"compiles and maintains"`, `"coordinates and tracks permits"` → `"building permit counter / review support staff"`.
2. **Account/admin check:** `matchedRoleTerms` contains `"account holder"` or `"administrator"` → `"applicant account holder or workflow administrator"`.
3. **Clerical check:** `matchedRoleTerms` contains `"clerk"`, `"technician"`, or `"coordinator"` → `"intake, review, or compliance coordinator"`.
4. **Submitter check:** `lower` contains `"applicant"`, `"submitter"`, `"preparer"`, `"operator"`, or `"respondent"` → `"applicant, submitter, or workflow operator"`.

Even `"staff"`, which is present in `ROLE_TERMS` at line 107, is never checked in `inferOperatorRole` or `inferBuyerRole`.

Run `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z` queried five private-sector operator verticals (`biggerpockets.com`, `quickbooks.intuit.com`, `community.shopify.com`, `thetruckersreport.com`, `aapc.com`). None of the 24 fetched documents used any of those 12 government-permitting nouns:

| Vertical (Sources in Run) | Actual Operator Nouns Present in Retained Excerpts | Matched `ROLE_TERMS` in Run | Resulting `operatorRoleGuess` |
| --- | --- | --- | --- |
| **BiggerPockets** (`001, 006, 007, 011, 017`) | `"landlord"`, `"property manager"`, `"PM"`, `"real estate investors"`, `"tenants"`, `"handy man"` | `[]` on all 5 | `"unknown operator role; needs direct validation"` |
| **QuickBooks Community** (`002, 008, 016, 024`) | `"bookkeeper"`, `"customers"`, first-person small-business billers (`"I have a recurring invoice template"`, `"We bill this month"`) | `[]` on all 4 | `"unknown operator role; needs direct validation"` |
| **Shopify Community** (`003, 009, 010, 013, 022`) | `"shopify store owners"`, `"store owners"`, `"supplier"`, first-person ecommerce bookkeepers (`"I have to manually enter into QB"`) | `["vendor"]` on `010` (`[]` on others) | `"unknown operator role; needs direct validation"` (`"vendor"` sets `buyerRoleGuess`, ignored by `inferOperatorRole`) |
| **TheTruckersReport** (`004, 012, 014, 018, 019`) | `"dispatcher"`, `"driver"`, `"DM"` (driver manager), `"fleet"` | `[]` on all 5 | `"unknown operator role; needs direct validation"` |
| **AAPC** (`005, 015, 020, 021, 023`) | `"biller"`, `"coder"`, `"CPC"`, `"providers"`, `"practice"`, `"Medicare Administrative Contractor (MAC)"` (`023` was HTTP 403) | `["department", "provider"]` on `005, 015`; `["provider"]` on `021` | `"unknown operator role; needs direct validation"` (`"department"`/`"provider"` only set `buyerRoleGuess`) |

**Compounding effect on `workflowReality` confidence:**
In [AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L1769-L1788), `matchedRoleTerms.length > 0` contributes `+0.14` to `rawSignal.confidence`. When `matchedRoleTerms` is empty (`+0.00`) and `matchedSeedTerms` is empty (`+0.00`, because the default seed `"SaaS workflow evidence"` only checks literal tokens `["saas", "workflow", "evidence"]`), any forum post that also misses `WORKFLOW_TERMS` (`002, 004, 008, 012, 019`) ends up at `confidence = 0.18 + 0.22 = 0.40 < 0.50`, leaving `workflowReality = "unresolved"`.

---

#### What It Would Take to Fix Role Inference for Non-Government Sources

1. **Separate domain-specific operator & buyer role lexicons in [AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L90-L111) and `inferOperatorRole` / `inferBuyerRole` ([AllowlistHttpSourceAdapter.ts](file:///c:/Users/dgreg/.gemini/antigravity/scratch/goose-worker-aven/src/adapters/sources/AllowlistHttpSourceAdapter.ts#L1735-L1767)):**
   - **Property management:** operator terms `["property manager", "leasing agent", "maintenance coordinator", "self-managing landlord", "landlord"]` → operator role `"property manager or self-managing landlord"`; buyer role `"property management firm or rental portfolio owner"`.
   - **Accounting / bookkeeping / billing:** operator terms `["bookkeeper", "accountant", "billing specialist", "accounts receivable", "accounts payable", "controller"]` → operator role `"bookkeeper or accounting operations staff"`; buyer role `"accounting practice or SMB finance owner"`.
   - **Ecommerce / retail operations:** operator terms `["store owner", "merchant", "ecommerce manager", "fulfillment coordinator", "inventory manager"]` → operator role `"ecommerce merchant or operations manager"`; buyer role `"ecommerce merchant or retail business owner"`.
   - **Trucking / logistics / dispatch:** operator terms `["dispatcher", "driver manager", "fleet manager", "load planner", "freight broker", "carrier", "driver"]` → operator role `"dispatcher, fleet coordinator, or carrier operator"`; buyer role `"motor carrier, fleet owner, or freight brokerage"`.
   - **Healthcare coding & revenue cycle:** operator terms `["medical coder", "coder", "biller", "billing specialist", "cpc", "practice manager", "revenue cycle"]` → operator role `"medical coder, biller, or revenue cycle specialist"`; buyer role `"medical practice, billing company, or healthcare provider"`.
2. **Recognize first-person operator workflow narratives in `operator_pain` / `public_discussion` forum threads:**
   - In forum posts (`QuickBooks`, `Shopify`, `BiggerPockets`), the author often writes in the first person (`"I have a recurring invoice template... requires a manual edit"`, `"I have to manually enter into QB"`, `"We bill this month..."`) without naming their job title in the third person.
   - When `sourceCategory === "operator_pain" || sourceCategory === "public_discussion"` and the text matches first-person operational patterns (`/\b(i|we)\s+(manually|bill|invoice|reconcile|sync|dispatch|schedule|submit|file|manage|track|export)\b/i`), combine that with the domain/vertical context (from the host or seed/vertical config) to infer a concrete operator role rather than returning `"unknown operator role; needs direct validation"`.
3. **Stop mapping generic words (`"vendor"`, `"provider"`, `"department"`, `"agency"`) to a single government buyer label in `inferBuyerRole`:**
   - Currently, any mention of `"department"` or `"provider"` in a healthcare article (like AAPC's `"provider outreach and education department"`) classifies `buyerRoleGuess` as `"public agency, county department, or compliance owner"`, and any mention of `"vendor"` in a Shopify app thread classifies `buyerRoleGuess` as `"regulated organization or operating entity"`. Scoping buyer inference by vertical prevents private-sector forum threads and generic `.gov` pages from being mislabeled.
