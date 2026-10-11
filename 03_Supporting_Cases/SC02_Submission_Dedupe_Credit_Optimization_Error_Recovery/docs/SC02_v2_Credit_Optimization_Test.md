# SC02 v2 — Credit Optimization Test

Scenario: `SC02_v2_Credit_Optimization`

Date: 2026-10-10

## Goal

Reduce unnecessary module execution while preserving correct workflow behavior.

## Optimization

The Update a Row route now runs only when:

- Search Rows returns a Row number; and
- the incoming score differs from the existing score.

## Test Results

| Test | Input | Expected Result | Actual Result | Operations | Credits | Status |
|---|---|---|---|---:|---:|---|
| OPT01 — Before | Existing ID, same score | Update runs | Update the existing row with the same value: SUB-CREDIT-001, HW-001, Nguyen An, 6A, 8 | 3 | 3 | PASS |
| OPT02 — After | Existing ID, same score | Update does not run | Search found the existing row; both Add and Update were skipped, and the row stayed unchanged | 2 | 2 | PASS |
| OPT03 — Changed | Existing ID, new score | Update runs and changes score to 9 | Update the existing row with the changed score: SUB-CREDIT-001, HW-001, Nguyen An, 6A, 9 | 3 | 3 | PASS |

## Result

- Operations saved per unchanged request: 1
- Credits saved per unchanged request: 1
- Estimated saving for 1,000 unchanged requests: 1*1,000 = 1,000 credits

## Known Limitations

- The optimization compares only the Score field.
- Changes to other fields are not detected when the score remains unchanged.
- The test uses simulated data and a small dataset.
- Concurrent writes from other users or scenarios are not tested.

## Evidence

- [Before optimization](../evidence/v2/04_Credit_Before.png)
- [After optimization](../evidence/v2/05_Credit_After.png)
- [Changed-score update](../evidence/v2/06_Changed_Score_Update.png)

The screenshots retain the original session/scenario names as historical evidence. OPT01 is the pre-optimization baseline; the exported v2 blueprint represents the optimized configuration. The projected 1,000-credit saving is an extrapolation, not a 1,000-request test. The reduction from 3 to 2 credits is 33.3%.
