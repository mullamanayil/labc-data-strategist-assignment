# Label Configuration Proposal

> Map the scanned content to the four labels (Public / Internal / Confidential / Restricted)
> and the eight descriptors. Propose auto-labelling rules and Copilot scoping for go-live.
> Ground each choice in the provided deck.

## Label mapping

| Label        | Descriptor | Note                                                                                                                                         |
| ------------ | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Public       | N/A        | External sharing allowed; Copilot can quote freely; Don't include PII, financial data, drafts, or unapproved release and security datas      |
| Internal     | N/A        | External sharing blocked and external email sharing is prompted; Don't include personal info, HR, unpublished financials, vendor negotiation |
| Confidential | People     | HR records, employee files, contact lists, SIN/PII bundles                                                                                   |
| Confidential | Financial  | Budgets, invoices, expense claims, receipts, financial statements                                                                            |
| Confidential | Commercial | Live procurement, bid evaluations, vendor contracts under negotiation                                                                        |
| Confidential | Audit      | Audit reports, compliance reviews, investigations                                                                                            |
| Confidential | Security   | Incident reports, threat assessments, network diagrams                                                                                       |
| Restricted   | Legal      | Closed-door committee, in-camera, Legal advice and privileged communications                                                                 |

## Auto-labelling rules

> Label Logic

| Trigger                                                                                                                                                                | Auto-label                       | Note                                                                |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- | ------------------------------------------------------------------- |
| Content mentions closed-door, in camera, leadership, privileged, committee, legal, senior, legal opinion                                                               | Suggest Restricted               | Extremely sensitive content for named people only                   |
| Content has PII detections, HR, payroll, benefits, employee, personnel, SIN, legal, vendor, contract, procurement, budget, invoice, expense, audit, incident, security | Suggest Confidential             | Sensitive personal, business, legal, financial, or security content |
| Current label is Public, no PII detections, and no major access concern                                                                                                | Public                           | Safe to share publicly                                              |
| Saved in HR site library                                                                                                                                               | Auto-apply Confidential - People | HR Benefits/Payroll sites should not rely on user confirmation      |
| None of the above rules apply                                                                                                                                          | Suggest Internal                 | Routine Assembly-only content                                       |

> Descriptor Logic:

| Trigger                                              | Auto-label                                                                   | Note                                                      |
| ---------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------- |
| HR, benefits, employee, personnel, payroll, SIN, PII | People                                                                       | Identifiable person or HR/personal information            |
| legal, counsel, privileged                           | Legal                                                                        | Legal advice or privileged material                       |
| committee, proceeding, in camera                     | Proceedings                                                                  | Committee or pre-tabling/proceedings material             |
| vendor, procurement, contract                        | Commercial                                                                   | Procurement, vendor, or commercial material               |
| budget, invoice, expense, financial                  | Financial                                                                    | Budgets, invoices, expenses, or financial records         |
| audit, review, investigation                         | Audit                                                                        | Audit, review, compliance, or investigation findings      |
| security, incident, network, threat                  | Security                                                                     | Security, incident response, or attack-useful information |
| MLA, member, constituency                            | Member Support                                                               | Services or files related to MLAs or constituency work    |
| Blank                                                | Recommended label is Public or Internal, or no descriptor keywords are found | No descriptor required                                    |

## Copilot and agent scoping

| Label        | Copilot Agent Scope         | Note                                                                       |
| ------------ | --------------------------- | -------------------------------------------------------------------------- |
| Public       | In scope                    | Safe for Copilot to reference freely                                       |
| Internal     | In scope for Assembly users | Safe if label is correct, no PII, and no anonymous links                   |
| Confidential | Limited in scope            | Not safe for broad rollout until audience, sharing, and label are verified |
| Restricted   | Excluded                    | Remove from Copilot scope entirely                                         |

## Sample-document labels

| Document                    | Label        | Descriptor  | Reason                                                                                                                             |
| --------------------------- | ------------ | ----------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| leadership_pack_draft.txt   | Restricted   | Proceedings | Five named leaders only, draft financials, in-camera committe context, and sensitive staffing change match the Restricted scenario |
| procurement_card_review.txt | Confidential | Audit       | Unfinalized review findings and procurment-card spending controls create serious trust risk if leaked                              |
| media_release_approved.txt  | Public       | N/A         | Approved for release and sheculed for the public assembly website                                                                  |
