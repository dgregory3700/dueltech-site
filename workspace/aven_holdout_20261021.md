# Aven holdout: run 20261021T134618Z

Date: 2026-11-24
For: Antigravity and the operator
From: Claude, independent reviewer
Data file: `workspace/aven_holdout_20261021.json` (same labels, machine-readable, full source IDs)
Companion: `workspace/aven_yardstick_20261028.md`

## What this is

A second labeled run, kept apart from the yardstick. A change is designed against the yardstick and then checked once against this holdout. If a change is tuned until the holdout passes, it stops being a holdout.

Labels were made from the title and retained excerpt in the archived source-queue objects (`goose-worker-aven` at `aa12d39`). Live pages were not re-fetched.

| Label | Meaning | Count |
| --- | --- | --- |
| Surface | Should reach the operator's review list (Park) | 0 |
| Borderline | Either outcome is acceptable | 0 |
| Do not surface | Should not reach the review list | 19 |
| Too thin | Not enough retained text to judge; should stay out, with an extraction reason | 5 |

## This holdout tests the noise side only

No source in this run is first-person operator testimony. The run returned 13 vendor or service-provider marketing pages, 4 GitHub issues about unrelated software, 1 state Medicaid instruction page, 1 template page and 5 blocked pages. Two of the blocked pages are Reddit threads whose titles suggest they were the relevant ones.

So this holdout can show that a change lets noise through. It cannot show that a change finds real leads. Run `20261028T222943Z` is the only archived run with forum threads, which means a holdout for the "finds real leads" side has to come from a future operator-authorized forum run.

## How Aven scores against it

| Version | Do-not-surface items reaching review (of 19) |
| --- | --- |
| `main` at `aa12d39` | 7 |
| `main` with every source given a known operator role (simulation) | 15 |
| `exp/connector-intake-refinements`, per the regenerated replay | 15 |

All 7 that reach review on `main` today are vendor marketing pages archived as `operator_pain` or `public_discussion`. The simulation shows that fixing role inference alone would more than double that.

## Pass criteria

1. None of the 19 "Do not surface" sources reach Park.
2. The 5 "Too thin" sources stay out of the review list and carry a reason that names the extraction problem.
3. The change must not name these sites, page titles or source IDs.

## Defects this run makes visible

- **Vendor marketing is filed as operator pain.** Of the 13 vendor or service-provider pages, 10 are archived as `operator_pain` and 2 as `public_discussion`, all quality B. Only 1 is archived as `competitor_vendor`.
- **Unrelated GitHub issues are filed as operator pain.** Sources 009 and 010 are bug reports for other software projects.
- **Roles come from marketing vocabulary.** The 7 pages that reach review are the ones where sales copy happened to contain a role word. Their archived operator roles are the generic government-era labels (clerk or coordinator, account holder or administrator, applicant or submitter).
- **The relevant pages were the blocked ones.** Sources 004 and 005 are Reddit threads that returned "Blocked".
- **The search returned the wrong kind of page.** This run and the yardstick run used the same seed. This one returned mostly vendor blogs; the yardstick run returned forum threads. That difference comes from the queries, not the gates.

## The 24 sources

| # | ID | Source | Label | Why | Archived category / quality | Archived validation |
| --- | --- | --- | --- | --- | --- | --- |
| 001 | `F3346DDBD0` | [Claim Resubmission and Reconsideration Process](https://www.azahcccs.gov/PlansProviders/RatesAndBilling/FFS/claimresubmission.html) | **Do not surface** | State Medicaid instructions for resubmitting claims. Official documentation, not operator testimony. | official_documentation / A | No Candidate |
| 002 | `DCF12F9B45` | [How Property Managers Track Maintenance Requests](https://taskestate.com/guide/how-property-managers-track-maintenance-requests) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | Park |
| 003 | `D95264001C` | [Issue record card: restore the controls when an edit is refused · Issu](https://github.com/mflr0/Solstice/issues/123) | **Do not surface** | GitHub issue about an unrelated software project. Not a business workflow. | secondary_commentary / D | No Candidate |
| 004 | `D39789498A` | [Blocked](https://www.reddit.com/r/automation/comments/1r70ymg/small_business_owners_what_repetitive_task_is/) | **Too thin** | Blocked or forbidden; nothing retrieved. The Reddit thread title suggests it would be relevant. | secondary_commentary / E | No Candidate |
| 005 | `FEC16DC963` | [Blocked](https://www.reddit.com/r/FreightBrokers/comments/1siez7c/is_it_just_me_or_is_the_manual_math_in_freight/) | **Too thin** | Blocked or forbidden; nothing retrieved. The Reddit thread title suggests it would be relevant. | secondary_commentary / E | No Candidate |
| 006 | `B7C42C2A32` | [What Is Invoice Processing? Steps & Best Practices](https://ramp.com/blog/accounts-payable/invoice-processing-explained) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | No Candidate |
| 007 | `43539CA92A` | [Digitizing Property Maintenance: Moving from Spreadsheets to CMMS](https://oxmaint.ai/industries/property-management/digitizing-property-maintenance-spreadsheets-to-cmms) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | Park |
| 008 | `664643BBB0` | [GitHub · Where software is built](https://github.com/tswarren/DepartureDesk/issues/214) | **Too thin** | GitHub returned an access message; nothing retrieved. | secondary_commentary / D | No Candidate |
| 009 | `F51D0D1B4D` | [Messaging /retry rewrites the transcript before admission, so any refu](https://github.com/NousResearch/hermes-agent/issues/139257) | **Do not surface** | GitHub issue about an unrelated software project. Not a business workflow. | operator_pain / B | No Candidate |
| 010 | `9105F3CC33` | [[Bug] OpenCodex 2.88.0: existing threads fail after model/provider swi](https://github.com/lidge-jun/opencodex/issues/7369) | **Do not surface** | GitHub issue about an unrelated software project. Not a business workflow. | operator_pain / B | No Candidate |
| 011 | `C87D226F93` | [A friendly error with "Try again" · Issue #99 · dwest1507/eliza-assess](https://github.com/dwest1507/eliza-assessment/issues/99) | **Do not surface** | GitHub issue about an unrelated software project. Not a business workflow. | secondary_commentary / D | No Candidate |
| 012 | `287BA36836` | [Property Management Software vs. Spreadsheets (2026)](https://pickspace.com/blog/property-management-software-vs-spreadsheets) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. It quotes a survey figure, which does not make it testimony. | public_discussion / B | Park |
| 013 | `1E6EA26495` | [Reduce Manual Data Entry in Your Business](https://columbiatechgroup.com/solutions/replace-manual-work) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | public_discussion / B | Park |
| 014 | `30C1CB4ED7` | [Forbidden](https://support.simplepractice.com/hc/en-us/articles/360016456811-Resolving-claim-rejections) | **Too thin** | Blocked or forbidden; nothing retrieved. | secondary_commentary / E | No Candidate |
| 015 | `43703EEAF5` | [Document Automation for Freight Ops: Stop Manual BOL Entry](https://www.laneproof.com/blog/document-automation-freight-ops-bol-data-entry) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | Park |
| 016 | `A3A3F2FFB3` | [How to Handle Claim Denials and Resubmit Successfully - Providers Care](https://providerscarebilling.com/how-to-handle-claim-denials-and-resubmit/) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | No Candidate |
| 017 | `798E3C6F06` | [How to Stop Wasting Hours on Manual Data Entry](https://www.easydatasync.com/stop-manual-data-entry) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | No Candidate |
| 018 | `9ACB3289C6` | [Freight Broker Document Management](https://www.brokerpro.com/posts/freight-broker-document-management/) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | Park |
| 019 | `0BA66C65FB` | [Property Maintenance Log - Spreadsheet Template](https://documentero.com/templates/real-estate/spreadsheet/property-maintenance-log/) | **Do not surface** | Template download page. No operator, no pain described. | operator_pain / B | No Candidate |
| 020 | `14B600478C` | [Forbidden](https://www.mhcautomation.com/blog/common-accounts-payable-issues-and-how-to-solve-them/) | **Too thin** | Blocked or forbidden; nothing retrieved. | secondary_commentary / E | No Candidate |
| 021 | `F117035F3D` | [Accounts Payable Errors & 7 Ways to Avoid Them](https://planergy.com/blog/7-ways-reduce-errors-accounts-payable-ap/) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | No Candidate |
| 022 | `F51717D915` | [Automated Document Generation for Freight Brokers](https://arktms.com/blog/automated-document-generation-freight-broker-tms) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | Park |
| 023 | `EAD96C3C03` | [How to resubmit denied insurance claims in private practice](https://www.gethealthie.com/blog/how-to-deal-with-denied-insurance-claims) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | competitor_vendor / C | No Candidate |
| 024 | `E08C6AF51F` | [What is Invoice Processing?](https://www.sage.com/en-us/blog/what-is-invoice-processing/) | **Do not surface** | Vendor or service-provider marketing content, not operator testimony. Competitor evidence at most. | operator_pain / B | No Candidate |
