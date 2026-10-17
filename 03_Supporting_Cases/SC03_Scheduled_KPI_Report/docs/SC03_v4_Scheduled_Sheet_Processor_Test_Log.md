# SC03 v4 — Scheduled Sheet Processor Test Log

Date: 2026-10-16

## Purpose
Read fictional class data from Google Sheets, validate and classify the records, write the results back to Google Sheets, and run the workflow on a schedule.

## Workflow
Manual Trigger / Schedule Trigger
→ Read KPI Input
→ Validate Records
→ Calculate and Classify KPI
→ Write KPI Report or Invalid Record

## Test Results

| Test | Expected Result | Actual Result | Status |
|---|---|---|---|
| T01 — Read input | Google Sheets returns 4 items | The output of Read KPI input node shows 4 items | PASS  |
| T02 — Valid processing | KPI_Report receives 3 rows | KPI_Report received 3 new rows for CLS-001, CLS-002 and CLS-003  | PASS  |
| T03 — Invalid processing | Invalid_Records receives CLS-004 | Invalid_Records received 1 new row for CLS-004 with kpi_status = INVALID_DATA | PASS |
| T04 — KPI classification | 60=AT_RISK; 80=WATCH; 95=ON_TRACK | CLS-003 (60% completion_rate) => kpi_status = AT_RISK; CLS-001 (80% completion_rate) =>kpi_status = WATCH; CLS-002 (95% completion_rate) => kpi_status = ON_TRACK | PASS |
| T05 — Scheduled execution | Workflow runs automatically without manual execution | A successful automatic execution was recorded during the temporary 1-minute schedule test; 3 report rows and 1 invalid row were appended | PASS |

## Schedule
- Timezone: Asia/Ho_Chi_Minh
- Frequency: Weekly
- Day: Friday
- Time: 08:00

## Security
- The spreadsheet contains fictional data only.
- Credential values and access tokens are not included in evidence.
- The spreadsheet remains private.

## Known Limitations
- Append Row may create duplicate rows when the workflow runs again.
- Duplicate prevention is not implemented in this version.
- KPI thresholds are fixed.
- The workflow does not send notifications when an execution fails.

## Evidence
- [KPI input](../evidence/v4/01_KPI_Input_Sheet.png)
- [Four input items](../evidence/v4/02_Read_Four_Items.png)
- [Output sheets](../evidence/v4/03_Report_and_Invalid_Results.png)
- [Scheduled settings](../evidence/v4/04_Schedule_Settings.png)
- [Scheduled execution](../evidence/v4/05_Scheduled_Execution.png)
- [Final workflow](../evidence/v4/06_Final_Workflow.png)

## Workflow Export
- [Download workflow JSON](../workflows/SC03_v4_Scheduled_Sheet_Processor.json)

## Blocker
None.