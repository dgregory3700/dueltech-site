# Aven yardstick: run 20261028T222943Z

Date: 2026-11-23
For: Antigravity and the operator
From: Claude, independent reviewer
Data file: `workspace/aven_yardstick_20261028.json` (same labels, machine-readable, full source IDs)

## What this is

A labeled list of the 24 sources Aven fetched in run `OE-CONTROLLED-LIVE-RESEARCH-20261028T222943Z`. Each source is marked with the outcome a careful human reviewer would want. Any change to role inference, categorization or the gates should be scored against this list before it is merged.

Labels were made from the title and retained excerpt in the archived source-queue objects (`goose-worker-aven` at `9d94995`). Live pages were not re-fetched, so the labels judge what Aven actually captured.

| Label | Meaning | Count |
| --- | --- | --- |
| Surface | Should reach the operator's review list (Park) | 7 |
| Borderline | Either outcome is acceptable | 2 |
| Do not surface | Should not reach the review list | 12 |
| Too thin | Not enough retained text to judge; should stay out, with an extraction reason | 3 |

## How Aven scores against it today

| Version | Surface items reaching review (of 7) | Do-not-surface items reaching review (of 12) |
| --- | --- | --- |
| Archived run (old routing review) | 4 | 6 |
| `main` after the one-decision fix, as reported | 0 | 0 |
| `exp/connector-intake-refinements`, as reported | 0 | 3 |

The experimental branch parks four sources: 005, 010, 015 and 021. Three are trade newsletters labeled "Do not surface" and one is borderline. It parks none of the seven real items.

## Pass criteria for any proposed change

1. All 7 "Surface" sources reach Park.
2. None of the 12 "Do not surface" sources reach Park.
3. The 3 "Too thin" sources stay out of the review list and carry a reason that names the extraction problem.
4. Each "Surface" source carries an operator role that matches the "Expected role" column in meaning. A government label such as "public agency, county department" on a private-sector source is a failure.
5. The change must not name these sites, thread titles or source IDs. A rule that only works on these 24 sources does not pass.

## Defects this run makes visible

- **The strongest lead was rejected by a keyword.** Source 022 says a tool is needed "so you don't have to re-enter them manually". `inferClaimResult` treats the substring "re-enter" as evidence that work removal is contradicted, so the description of the pain counted against it.
- **Forum posts are classified as vendor pages.** Sources 001, 002 and 007 are first-person forum posts archived as `competitor_vendor`, quality C, which makes them not source-grade.
- **Vendor and trade-press pages are classified as operator pain.** Source 016 is QuickBooks' own how-to page. Sources 005, 015 and 021 are AAPC newsletter articles. All four are archived as `operator_pain`, quality B.
- **No operator role on any of the 7 real items.** All 24 sources have an unknown operator role.
- **One vertical retrieved the wrong meaning.** The five trucking threads (004, 012, 014, 018, 019) are disputes between drivers and dispatchers, not dispatch workflow problems. That is a query problem, not a gate problem.
- **Extraction lost the post on some pages.** Source 008 retained page chrome instead of the question. Sources 002 and 019 retained 53 and 43 characters.

## Limits

- One run, 24 sources, one labeler. This is a regression check. It does not prove that a change is good in general.
- The operator can overrule any label. Change the JSON file and note the reason.
- A second labeled run is needed as a holdout before any role-inference change is merged. Run `20261021T134618Z` is the suggested one.

## The 24 sources

| # | ID | Source | Label | Expected role | Why | Archived category / quality | Archived routing |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 001 | `8C9108525D` | [How do you all track and manage tenant maintenance requests?](https://www.biggerpockets.com/forums/925/topics/1252240-how-do-you-all-track-and-manage-tenant-maintenance-requests) | **Surface** | self-managing landlord | First-person: requests arrive by text, call and email and are hard to track. Poster is also building a tool, so read as pain plus a would-be competitor. | competitor_vendor / C | Reject |
| 002 | `1031638C34` | [Automatically enter the following month on invoices as part of product](https://quickbooks.intuit.com/learn-support/en-us/reports-and-accounting/automatically-enter-the-following-month-on-invoices-as-part-of/00/1229978) | **Too thin** |  | Retained excerpt is one sentence (53 characters). Title suggests a recurring manual invoice edit, but there is not enough text to judge. | competitor_vendor / C | Reject |
| 003 | `D40628310A` | [How are you managing bookkeeping alongside your Shopify store as it gr](https://community.shopify.com/t/how-are-you-managing-bookkeeping-alongside-your-shopify-store-as-it-grows/608349) | **Surface** | Shopify merchant doing own bookkeeping | Reconciling Shopify, payment processors, bank and accounting software as the store grows; spreadsheets in use. | public_discussion / B | Park |
| 004 | `5BE015C6EA` | [dispatching services](https://www.thetruckersreport.com/truckingindustryforum/threads/dispatching-services.2440667/) | **Do not surface** |  | Asks whether a dispatching service is legitimate. Not a workflow problem. | secondary_commentary / E | Reject |
| 005 | `52C61A1B46` | [Sidestep Duplicate Claim Denials by Properly Appending Repeat Modifier](https://www.aapc.com/codes/coding-newsletters/my-cardiology-coding-alert/claims-sidestep-duplicate-claim-denials-by-properly-appending-repeat-modifiers-172134-article) | **Do not surface** |  | Trade newsletter advice on claim modifiers. Not operator testimony; the issue is coding knowledge, not repeated manual work. | operator_pain / B | Park |
| 006 | `46B24CB8CC` | [How do you track maintenance requests for your properties](https://www.biggerpockets.com/forums/90/topics/524768-how-do-you-track-maintenance-requests-for-your-properties) | **Surface** | landlord or small property manager | Spreadsheet tracking with 'lots of manual work', the steps listed in order, and an explicit wish to automate them. Strongest item in the run. | operator_pain / B | Park |
| 007 | `4F54E4EBD0` | [Property Management Spreadsheet Recommendations](https://www.biggerpockets.com/forums/899/topics/1111762-property-management-spreadsheet-recommendations) | **Borderline** | landlord with 40+ rentals | Real operator at scale, but asks for a spreadsheet and says free is preferred. No pain described. | competitor_vendor / C | Reject |
| 008 | `6407E2BA5E` | [Is there a way to upload my invoices from the beginning of the year un](https://quickbooks.intuit.com/community/reports-and-accounting-5/is-there-a-way-to-upload-my-invoices-from-the-beginning-of-the-year-until-now-without-manually-entering-each-one-38447) | **Too thin** |  | Extraction captured page chrome, not the post. Title describes bulk invoice entry by hand, so the page may be relevant. | secondary_commentary / E | Reject |
| 009 | `D5F07008BA` | [Shopify store owners — what repetitive task is still taking up your ti](https://community.shopify.com/t/shopify-store-owners-what-repetitive-task-is-still-taking-up-your-time/704258) | **Surface** | Shopify merchant | Merchants list repetitive tasks: reconciling inventory across channels, spreadsheets, chasing missing information, failed integrations forcing manual work. | public_discussion / B | Park |
| 010 | `8E3608361C` | [Calculating fulfilled orders by date for accounting purposes](https://community.shopify.com/c/accounting-and-taxes/calculating-fulfilled-orders-by-date-for-accounting-purposes/td-p/1508153) | **Borderline** | Shopify merchant | Needs an accurate fulfilled-by-date report for accounting. Narrow, and existing report apps are already suggested in the thread. | public_discussion / B | Park |
| 011 | `08623C1BC3` | [How do your tenants submit maintenance request?](https://www.biggerpockets.com/forums/899/topics/1060114-how-do-your-tenants-submit-maintenance-request) | **Do not surface** |  | One rental home, wants a free tool, asks whether they are overthinking it. | secondary_commentary / D | Reject |
| 012 | `31BA6587D5` | [Dispatcher Problems? Maybe?](https://www.thetruckersreport.com/truckingindustryforum/threads/dispatcher-problems-maybe.2458663/) | **Do not surface** |  | New driver unsure about a dispatcher. Personal dispute, not a workflow. | secondary_commentary / D | Reject |
| 013 | `17D3CE419F` | [Bookkeeping; is there an easier way?](https://community.shopify.com/t/bookkeeping-is-there-an-easier-way/371583) | **Surface** | Shopify merchant doing own bookkeeping | First-person, monthly: 'I have to manually enter into QB' installments and gift cards, from a Shopify report. Asks for the best sync app. | public_discussion / B | Park |
| 014 | `43A1016733` | [Forced dispatch](https://www.thetruckersreport.com/truckingindustryforum/threads/forced-dispatch.245075/) | **Do not surface** |  | Driver story about forced dispatch. Not a workflow. | secondary_commentary / D | Reject |
| 015 | `2FF8140F22` | [Fix Your Duplicate Claim Denial Problem with Modifiers : Compliance](https://www.aapc.com/codes/coding-newsletters/my-general-surgery-coding-alert/compliance-fix-your-duplicate-claim-denial-problem-with-modifiers-172567-article) | **Do not surface** |  | Same newsletter content as 005. | operator_pain / B | Park |
| 016 | `13E84E92DC` | [How to schedule recurring invoices](https://quickbooks.intuit.com/r/invoicing/how-to-schedule-recurring-invoices/) | **Do not surface** |  | QuickBooks' own how-to page for recurring invoices. Vendor content showing the incumbent already covers this; competitor evidence at most. | operator_pain / B | Park |
| 017 | `DBF01F66DC` | [Help! How do you do Tenant Tracking??](https://www.biggerpockets.com/forums/899/topics/1198074-help-how-do-you-do-tenant-tracking) | **Surface** | self-managing landlord | First-person: tracking tenants 'via paper and notes', asking for the best system for private self-managers. Short but clear. | secondary_commentary / D | Reject |
| 018 | `AF446C24FC` | [Dealing with Dispatch (for newbies)](https://www.thetruckersreport.com/truckingindustryforum/threads/dealing-with-dispatch-for-newbies.273839/) | **Do not surface** |  | Driver anecdotes about dispatch. Not a workflow. | secondary_commentary / D | Reject |
| 019 | `BA76DB96E9` | [My dispatcher actually said this](https://www.thetruckersreport.com/truckingindustryforum/threads/he-actually-said-this.2538149/) | **Do not surface** |  | 43 characters retained; thread is a driver complaint about a dispatcher. | operator_pain / B | Park |
| 020 | `80F2AF689F` | [What is insurance claim denials and re-submission](https://www.aapc.com/discuss/threads/what-is-insurance-claim-denials-and-re-submission.150217/) | **Do not surface** |  | Generic one-line question about claim denial best practices. No operator detail. | secondary_commentary / E | Reject |
| 021 | `A783238AED` | [Follow This Advice for Clean Medicare Claims : Compliance](https://www.aapc.com/codes/coding-newsletters/my-general-surgery-coding-alert/compliance-follow-this-advice-for-clean-medicare-claims-173609-article) | **Do not surface** |  | Trade newsletter advice on Medicare claim codes. Not operator testimony. | operator_pain / B | Park |
| 022 | `DAEFD2F39F` | [What app works best for posting purchase orders in to Xero?](https://community.shopify.com/t/what-app-works-best-for-posting-purchase-orders-in-to-xero/563000) | **Surface** | Shopify merchant or inventory/purchasing staff | Purchase orders received in Stocky must be re-entered into Xero as bills; paid tools named as options. Clear double entry with evidence people pay to remove it. | public_discussion / B | Reject |
| 023 | `8114DC68CA` | [Forbidden](https://www.aapc.com/blog/28051-resubmitting-claims-can-get-you-in-trouble/) | **Too thin** |  | HTTP 403. Nothing retrieved. | secondary_commentary / E | Reject |
| 024 | `69024BB54D` | [How do I keep a consistent invoice number for select customers?](https://quickbooks.intuit.com/learn-support/en-us/reports-and-accounting/how-do-i-keep-a-consistent-invoice-number-for-select-customers/00/1491792) | **Do not surface** |  | Invoice numbering quirk in QuickBooks. A configuration question, not repeated manual work. | operator_pain / B | Park |
