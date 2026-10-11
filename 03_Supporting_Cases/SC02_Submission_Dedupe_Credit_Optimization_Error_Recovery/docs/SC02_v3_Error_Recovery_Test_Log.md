# SC02 v3 — Error Recovery Test Log

**Scenario:** `SC02_v3_Error_Recovery`  
**Test date:** October 11, 2026  
**Technical test status:** DONE — supplied blueprint and ten screenshots reviewed

## Purpose

Extend SC02 v2 with manual recovery of failed Google Sheets operations while retaining submission-ID lookup and the score-change filter. Failed work is saved rather than silently discarded.

## Configuration

- Flow: Webhook (1) → Search Rows (2) → Router (3) → Add a Row (4) or Update a Row (5).
- Test tab: `Credit_Test`; columns A–E: Submission_ID, Assignment_ID, Student_Name, Class, Score.
- Search compares column A to incoming `submission_id`; limit 1.
- Create filter: Search Row number does not exist.
- Update filter: Search Row number exists AND incoming score is numerically different from stored Score.
- Update Row number comes from Search; all five incoming values come from Webhook.
- Process data in order: Yes. Store incomplete executions: Yes.
- Retry error handlers on Search, Add and Update; Automatically complete execution: No.
- Exported steady-state configuration points all three Sheets modules to `Credit_Test`, not the deliberately missing tab.
- Public blueprint contains null connection/webhook references and a spreadsheet-ID placeholder. Users must configure their own resources after import.

## Test Inputs

```json
{
  "submission_id": "SUB-ER-001",
  "assignment_id": "HW-001",
  "student_name": "Nguyen An",
  "class_name": "6A",
  "score": 8
}
```

Use this base payload with the ID and score changes in the table below. Start with an empty test tab containing only its headers. Resolve each failure before starting the next test because sequential processing can block new runs while an incomplete execution remains unresolved.

## Results and Evidence

Evidence paths are relative to this document.

| Test | Action | Observed result | Status | Evidence |
|---|---|---|---|---|
| ER01 | Send SUB-ER-001, score 8 | Add ran; one row was created with score 8. | PASS | [Normal creation](../evidence/v3/03_Add_Success.png) |
| ER02 | Repeat SUB-ER-001, score 8 | Both write routes were skipped; the sheet remained at one row with score 8. | PASS | [Unchanged request](../evidence/v3/04_Optimize_Success.png) |
| ER03 | Set only Add's target tab to Credit_Test_Missing; send SUB-ER-002, score 8 | Add returned HTTP 400 INVALID_ARGUMENT, RuntimeError: unable to parse range. The error panel confirmed an incomplete execution was saved. No new row appeared. | PASS | [Add failure](../evidence/v3/05_Add_Failure.png) |
| ER04 | Correct the saved Add execution and resolve manually | Incomplete-execution attempt showed Manual / Success / 1 operation; SUB-ER-002 appeared once with score 8. | PASS | [Add recovery](../evidence/v3/06_Add_Recovery.png) |
| ER05 | Set only Update's target tab to Credit_Test_Missing; send SUB-ER-001, score 9 | Update returned HTTP 400 INVALID_ARGUMENT, RuntimeError. An incomplete execution was saved; stored score remained 8. | PASS | [Update failure](../evidence/v3/07_Update_Failure.png) |
| ER06 | Correct the saved Update execution and resolve manually | Attempt showed Manual / Success / 1 operation; SUB-ER-001 remained one row, now at score 9. | PASS | [Update recovery](../evidence/v3/08_Update_Recovery.png) |
| ER07 | Set only Search's target tab to Credit_Test_Missing; send SUB-ER-003, score 7 | Search returned HTTP 400 INVALID_ARGUMENT, RuntimeError. An incomplete execution was saved; SUB-ER-003 was absent. | PASS | [Search failure](../evidence/v3/09_Search_Failure.png) |
| ER08 | Correct the saved Search execution and resolve manually | Attempt showed Manual / Success / 2 operations; the final sheet included SUB-ER-003 once with score 7. | PASS | [Search recovery and final sheet](../evidence/v3/10_Search_Recovery.png) |

Supporting configuration evidence: [Canvas](../evidence/v3/01_Recovery_Scenario_Overview.png), [Settings](../evidence/v3/02_Recovery_Setting.png).

## Final Dataset

| Submission_ID | Assignment_ID | Student_Name | Class | Score |
|---|---|---|---|---:|
| SUB-ER-001 | HW-001 | Nguyen An | 6A | 9 |
| SUB-ER-002 | HW-001 | Nguyen An | 6A | 8 |
| SUB-ER-003 | HW-001 | Nguyen An | 6A | 7 |

`10_Search_Recovery.png` already contains the final dataset; a separate final-sheet screenshot is not required.

## Recovery Procedure

1. Inspect the failed module and its saved error.
2. Open the incomplete execution through Details or Resolve.
3. In that saved execution, correct the module's target tab to `Credit_Test` and save.
4. Run the saved execution from that screen using its preserved input. Do not resend the webhook as a substitute for recovery.
5. Check the manual attempt result and destination data.
6. Also restore the corresponding module in the main scenario so future requests use the correct tab.

The screenshots establish successful manual attempts and resulting data. They do not capture every edit click or independently prove that no additional Postman requests were sent.

## Findings and Limitations

- Three module failure locations were tested with the same deliberate configuration error. This is not coverage of three different error types.
- Automatic retry schedules, connection failures, rate limits and timeout recovery were not tested.
- Recovery starts at the failed module. Add recovery does not repeat the preceding ID search; concurrent writes or a write that succeeded before a lost response can still create duplicates.
- Update recovery uses a saved row location; another writer can move or change that row before recovery.
- Sequential processing serializes this scenario, not all external writers. Pending incomplete executions can delay later requests.
- Search limit 1 does not detect duplicate IDs already in the sheet.
- The optimization checks only Score; changes to other fields are ignored when Score is unchanged.
- Validation, normalization, conflict approval and a full duplicate-review process are not implemented.

## Reflection

Retrying unchanged settings does not repair a missing-tab configuration. Manual resolution corrects the saved failed operation and resumes its preserved data. Correcting the main scenario prevents the same configuration error from affecting future requests. Recovery success must be checked against the sheet, not only a webhook acknowledgement.
