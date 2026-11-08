# SC04 — Homework Submission Matching and Reporting

An n8n automation that turns homework-form responses into matched student records, routes uncertain matches for coordinator review, and updates student- and class-level reporting.

## Business problem

Homework submissions often contain inconsistent names. Matching each response manually delays reporting and can produce incorrect attempt counts or scores. This solution matches only unambiguous active student records; every uncertain case remains reviewable rather than being guessed.

## What the solution does

- Normalizes submitted name and class code.
- Matches each response against `Student_Master`.
- Automatically records exactly one active match as `AUTO_MATCHED`.
- Routes multiple, no-match, and inactive-student cases to `PENDING_REVIEW` with a reason.
- Lets a coordinator confirm, reject, or request resubmission through review fields.
- Reactivates an inactive selected student only after a valid confirmation.
- Records review and notification events, and prevents a duplicate resubmission email with `Notification_Log`.
- Builds `Student_Report` and `Class_Report` from resolved matches only.

## Workflow design

| Workflow | Trigger | Responsibility |
|---|---|---|
| `SC04_Intake_And_Match` | New `Form_Responses` row | Normalize input, evaluate exact-match candidates, then upsert the appropriate `Submissions` outcome. |
| `SC04_Review` | `Submissions.review_ready = TRUE` | Validate the coordinator decision, apply the approved outcome, update logs, and send a deduplicated resubmission email when needed. |

The full branch criteria and acceptance evidence are in [docs/SC04_Test_Plan.md](docs/SC04_Test_Plan.md).

## Repository structure

```text
.
├── README.md
├── workflows/
│   ├── SC04_Intake_And_Match.json
│   └── SC04_Review.json
├── data/
│   └── SC04_Sample_Data.xlsx
├── docs/
│   ├── Workflow_Charter.md
│   ├── RUNBOOK.md
│   ├── Google_Sheets_Report_Formulas.md
│   ├── SC04_Test_Plan.md
│   ├── 02_AS_IS_Process_Map_Homework_Submission.drawio
│   ├── 02_AS_IS_Process_Map_Homework_Submission.png
│   ├── 03_TO_BE_Process_Map_Homework_Submission.drawio
│   └── 03_TO_BE_Process_Map_Homework_Submission.png
└── evidence/
```

## Quick start

1. Create a Google Form and link its responses to a Google Spreadsheet, following [docs/RUNBOOK.md](docs/RUNBOOK.md).
2. Create the remaining tabs and headers with `data/SC04_Sample_Data.xlsx` as the template.
3. Copy the ready-to-use report formulas in [docs/Google_Sheets_Report_Formulas.md](docs/Google_Sheets_Report_Formulas.md).
4. Import both workflow JSON files into n8n.
5. Connect your own Google Sheets credentials to every Google Sheets node and Gmail credentials to `Send_Resubmission_Email`.
6. In every Sheets node, select your spreadsheet and target tab again; public templates intentionally contain no source spreadsheet ID or URL.
7. Set the actual Google Form URL in `Send_Resubmission_Email`.
8. Test with fictional data, then activate the two triggers only after validation.

See [docs/RUNBOOK.md](docs/RUNBOOK.md) for the full configuration and testing sequence.

## Public-safety note

This is a public-safe portfolio template. Workflow credentials, source spreadsheet IDs/URLs, n8n metadata, webhook IDs, personal email addresses, account profiles, and browser URLs have been removed or redacted. The included data uses fictional student records and a placeholder test address.

## Known limitations

- Name matching is exact after normalization; it does not use fuzzy matching.
- Google Sheets triggers use polling, so processing is not instant and depends on the configured interval.
- The XLSX file preserves sample structure and values; report formulas are designed for Google Sheets and should be configured or checked after upload.
- Email delivery is deduplicated by the notification record, but a failed send requires review of `Notification_Log` before retrying.
