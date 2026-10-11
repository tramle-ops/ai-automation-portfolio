# SC02 v1 — Dedupe Test Results

Scenario: `SC02_v1_Dedupe`

Date: 2026-10-08

## Business Rules

- Submission_ID identifies one submission.
- A new Submission_ID creates a row.
- An existing Submission_ID updates the matching row.
- For this exercise, incoming values replace existing values.
- Different submission IDs remain separate, even with the same name.

## Configuration

- Search column: Submission_ID
- Comparison: Equal to
- Search value: mapped submission_id from Webhook
- Search limit: 1
- No-match behavior: verified in TC01; a new Submission_ID reaches the Create route.
- Create filter: Search Row number does not exist
- Update filter: Search Row number exists
- Update Row number: mapped from Search Rows
- Process data in order: enabled.

## Test Results

| Test ID | Purpose | Expected result | Actual result | Status |
|---|---|---|---|---|
| TC01 | Create a new submission | 1 row; score 8; Add runs | Create a new row: SUB-DEDUPE-001, HW-001, Nguyen An, 6A, 8  | PASS |
| TC02 | Repeat the same payload | Still 1 row; score 8; Update runs | Update ran on the existing row; the sheet remained at one row with score 8 | PASS |
| TC03 | Change score for the same ID | Still 1 row; score 9; Update runs | Still 1 row, score is updated to 9, other columns stay the same | PASS |
| TC04 | New ID with the same name | 2 rows; Add runs | Create a new row: SUB-DEDUPE-002, HW-001, Nguyen An, 6A, 7 | PASS |

## Reflection

- Why is Student_Name unsuitable as the unique key? One student can submit multiple assignments or attempts, and different students may have the same name.
- How is Submission_ID different from Row number? Submission_ID is a business identifier received from the source system. Row number is the physical location of a record in Google Sheets and may change when rows are inserted, deleted, or reordered.
- Why do updated values come from the Webhook? The Webhook contains the latest incoming submission data, while Search Rows identifies which existing row must be updated.
- What happens if Search stops when no match is found? No bundle reaches the Create route, so the workflow cannot create a row for a new Submission_ID.

## Known Limitations

- Input validation is not implemented.
- Existing duplicate IDs are not detected because Search limit is 1.
- Other writers can still change the sheet.
- Incoming values overwrite existing values without approval.
- Changed IDs, letter case and extra spaces are not normalized.

## Evidence

| Test | Screenshot |
|---|---|
| Scenario | [Overview](../evidence/v1/01_Scenario_Overview.png) |
| Search configuration | [Page 1](../evidence/v1/02_Search_Settings_Page_1.png), [Page 2](../evidence/v1/02_Search_Settings_Page_2.png) |
| TC01 | [Create](../evidence/v1/03_Create_Result.png) |
| TC02 | [Repeat](../evidence/v1/04_Repeat_Request_Result.png) |
| TC03 | [Update](../evidence/v1/05_Update_Result.png) |
| TC04 | [Final sheet](../evidence/v1/06_Final_Sheet.png) |

The screenshots retain their original W3S03 scenario title as historical evidence. Renaming the exported scenario does not require rerunning these tests.
