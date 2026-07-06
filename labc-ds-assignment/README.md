# Senior Data Strategist — Take-Home Submission

Start with `ASSIGNMENT.md`. The sensitivity-label deck (`INFORMATION_PROTECTION.pptx`)
is provided alongside this repository.

## How to run my work

1. Open the Excel workbook in the `output/` folder.
2. Review the dashboard and `Risk_View` tab for the cleaned risk summary.
3. Review the Power Query logic used to clean and combine the data.
4. Review the `governance/` folder for the label configuration proposal and ethics memo.
5. Review `SLIDES.md` for the 3-slide presentation summary.

The analysis was completed using Excel Power Query and Excel tables. No separate Python or application setup is required.

To reproduce the Excel output:

1. Open the workbook.
2. Go to **Data > Queries & Connections**.
3. Open the Power Query queries.
4. Confirm the source file paths point to the `data/` folder in this repository.
5. Refresh all queries.

The main source files used are:

- `data/site_inventory.csv`
- `data/pii_detections.csv`
- `data/sharing_links.csv`
- `data/license_assignments.csv`
- `data/data_dictionary.csv`
- `INFORMATION_PROTECTION.pptx`

## Assumptions

- The data pack is a working draft and may not represent the full Microsoft 365 environment.
- The provided sensitivity-label deck is the source of truth for label decisions.
- I used the Assembly’s labels: Public, Internal, Confidential, and Restricted.
- I did not directly translate Assembly content into the BC Public Service Protected A/B/C scheme.
- Restricted content should be excluded from Copilot entirely.
- Confidential content may be available to Copilot only where the user already has access, and only after sharing and audience controls are reviewed.
- Public and Internal content can be in Copilot scope only when labels are accurate and no sensitive indicators are present.
- Anonymous links and external guest access are treated as risk indicators that should be reviewed before go-live.

## What I'd do with more time

- Validate the scan results with site owners and business areas.
- Confirm whether PII counts represent files, records, or detection matches.
- Review anonymous links and external sharing with site owners.
- Consult privacy, legal, HR, and security stakeholders before a full Copilot rollout
