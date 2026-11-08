# Runbook — SC04 Homework Submission Matching and Reporting

## 1. Create and configure the Google Form

Create a Google Form as a quiz so that Google writes the total score to the response sheet. In **Settings → Responses**, turn on **Collect email addresses** and choose verified email collection where available.

Create these required questions. Keep the question titles exactly as shown because the intake workflow maps those column names.

| Question title | Type | Required configuration |
|---|---|---|
| `student_name` | Short answer | Required. Students type their full name. |
| `class_code` | Dropdown | Required. Include active class codes only. |
| `assignment_id` | Dropdown | Required. Include only the current assignment ID, for example `HW-001`. |

For the quiz score, use Google Forms' native total-score field. Do not add a separate free-text question named `Score`.

### Link the Form to the response sheet

1. Open the Form's **Responses** tab and select **Link to Sheets**.
2. Choose **Select existing spreadsheet** and select the SC04 Google Spreadsheet.
3. Google Forms creates a response tab. Rename that tab to `Form_Responses` if needed.
4. Confirm that the first row contains these headers: `Timestamp`, `Email Address`, `student_name`, `class_code`, `assignment_id`, and `Score`.
5. Submit one fictional response and confirm that it appears as a new row in `Form_Responses`.

Do not manually rename or reorder these six columns after configuring n8n. If the current assignment changes, update the `assignment_id` dropdown before reopening the form for new submissions.

## 2. Prepare the spreadsheet

1. Upload `../data/sample_data.xlsx` to Google Drive and open it as a Google Spreadsheet, or create an equivalent spreadsheet.
2. Keep these tabs and headers: `Form_Responses`, `Student_Master`, `Submissions`, `Review_Log`, `Notification_Log`, `Student_Report`, and `Class_Report`.
3. Replace all sample records with fictional test data before testing. Keep `student_id`, `official_name`, `normalized_name`, `class_code`, and `active` in `Student_Master`.
4. Copy the formulas from [report_formulas.md](report_formulas.md) into `Student_Report` and `Class_Report`. The `.xlsx` export does not reliably preserve Google Sheets dynamic formulas.

## 3. Import and configure n8n

1. Import `../workflows/SC04_Intake_And_Match.json` and `../workflows/SC04_Review.json`.
2. Add your Google Sheets credential to every Google Sheets trigger and Google Sheets node.
3. Add your Gmail credential to `Send_Resubmission_Email`.
4. For every Google Sheets node, select your spreadsheet and its intended tab again. The exported templates deliberately use a placeholder spreadsheet URL and retain no source connection.
5. In `Send_Resubmission_Email`, replace `REPLACE_WITH_YOUR_HOMEWORK_FORM_URL` with the public URL of your own homework form.
6. Check that the intake trigger watches new rows in `Form_Responses`, and that `Watch_Review_Ready` watches `Submissions` updates.

Do not activate either workflow until the manual tests below pass.

## 4. Coordinator review procedure

1. Find a row where `match_status = PENDING_REVIEW`.
2. Complete the review fields as required for its `review_reason`.
3. Set `review_ready = TRUE`.
4. Wait for the review workflow, then confirm `review_ready` returned to `FALSE`.
5. Check `Submissions` and the relevant log. For a resubmission request, verify `Notification_Log` before retrying so that the email is not duplicated.

### Valid decisions

| Review reason | Valid decision | Required selection | Result |
|---|---|---|---|
| `MULTIPLE_MATCHES` or `NO_EXACT_MATCH` | `CONFIRMED` | An active student in the submitted class | `MANUALLY_MATCHED` |
| `MULTIPLE_MATCHES` or `NO_EXACT_MATCH` | `NOT_CONFIRMED` | None | `RESUBMISSION_REQUIRED` and one email attempt |
| `INACTIVE_STUDENT` | `CONFIRMED` | The matching inactive student | Reactivate and set `MANUALLY_MATCHED` |
| `INACTIVE_STUDENT` | `REJECTED` | None | `REJECTED` |

## 5. Test before activation

Run the 14 acceptance cases in [SC04_Test_Plan.md](SC04_Test_Plan.md). In particular, test all four intake branches, invalid review inputs, inactive-student reactivation, email deduplication, and both reports. Use only a controlled inbox and fictional records.

## 6. Operating checks

- Review `PENDING_REVIEW` cases regularly; unresolved submissions are excluded from reports.
- Investigate any `review_validation_error` before changing the same row again.
- For a failed email, inspect `Notification_Log.email_error`; resolve the cause before retrying.
- Check the polling interval and n8n execution history if an expected update is delayed.
