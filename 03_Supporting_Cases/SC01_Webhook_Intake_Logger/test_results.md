# Test Results — Webhook Intake Logger

| Test ID | Scenario | Expected Result | Actual Result | Status |
|---|---|---|---|---|
| TC01 | Valid payload | HTTP 200 and a complete row is added | HTTP 200 received and a complete row was added | PASS |
| TC02 | Number and boolean values | Data types are preserved | Score 7.5 and Valid FALSE were stored correctly | PASS |
| TC03 | Missing student_name | A row is logged with a blank name and the limitation is documented | HTTP 200 received and a row was added with a blank Student_Name | PASS |

## Known Limitation

The current version logs requests but does not reject a payload when student_name is missing. Input validation will be added in a later iteration.