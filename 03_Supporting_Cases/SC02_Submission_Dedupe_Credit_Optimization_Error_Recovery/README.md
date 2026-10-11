# Supporting Case 02 — Submission Dedupe, Credit Optimization and Error Recovery

**Author:** Tram Le  
**Tools:** Make, Google Sheets, Postman  
**Development dates:** October 8–11, 2026  
**Implemented scope:** Submission-ID duplicate prevention, score-based credit optimization, and manual error recovery  
**Status:** Completed — tested prototype for the implemented scope

## Business Problem

A fictional homework intake process receives new submissions, repeated requests and score corrections. Appending every request can create extra rows; updating unchanged scores consumes unnecessary credits. A failed lookup or write also requires a visible recovery path.

This package demonstrates three successive versions of the same intake pattern. They are alternatives in its development history, not three connected workflows to run together. Use v3 as the latest prototype; retain v1 and v2 to explain the tested improvements.

## Version History

| Version | Behavior | Evidence document |
|---|---|---|
| SC02_v1_Dedupe | Search by Submission_ID; create if not found; update if found, including unchanged requests. | [v1 tests](docs/SC02_v1_Dedupe_Test_Results.md) |
| SC02_v2_Credit_Optimization | Retain v1 lookup; update only when the incoming score differs from stored Score. | [v2 tests](docs/SC02_v2_Credit_Optimization_Test.md) |
| SC02_v3_Error_Recovery | Retain v2 logic; save failed Search/Add/Update executions for manual resolution. | [v3 tests](docs/SC02_v3_Error_Recovery_Test_Log.md) |

## Data and Rules

Input fields: `submission_id`, `assignment_id`, `student_name`, `class_name`, `score`.

Sheet columns A–E: `Submission_ID`, `Assignment_ID`, `Student_Name`, `Class`, `Score`.

- Submission_ID identifies a submission; different IDs remain separate even if student names match.
- Search uses exact text equality and returns at most one matching row.
- Create runs when the Search Row number does not exist.
- Update targets the Row number returned by Search and maps incoming values from Webhook.
- In v1, an existing ID always updates. In v2/v3, Update also requires a numeric Score difference.
- Incoming values overwrite existing fields when Update runs; approval is not implemented.
- In v3, sequential processing and incomplete-execution storage are enabled. Retry handlers on Search/Add/Update have automatic completion disabled.

## Evidence Summary

| Area | Verified or recorded result |
|---|---|
| v1 | Reviewed screenshots show new-row creation, unchanged-request update, changed-score update, and a separate row for a new ID with the same name. |
| v2 | Previously reviewed measurements: unchanged request uses 3 operations/credits before optimization and 2 after; changed score uses 3. |
| v3 | Reviewed blueprint and ten screenshots show normal creation, skipped unchanged writes, and saved failures followed by successful manual recovery at Add, Update and Search. |

The unchanged-score test saved **1 operation and 1 credit per request**, a **33.3% reduction** from the measured 3-credit baseline. An estimated 1,000 unchanged requests would save 1,000 credits at this rate; this is an extrapolation, not a load test or monetary ROI calculation. The v3 tests preserve skipped-write behavior but do not remeasure total credit costs of recovery.

The final v3 test sheet contains three unique IDs: SUB-ER-001 at score 9, SUB-ER-002 at score 8, and SUB-ER-003 at score 7. Its last recovery screenshot also serves as final-sheet evidence.

## Repository Contents

| Path | Purpose |
|---|---|
| `docs/SC02_v1_Dedupe_Test_Results.md` | v1 configuration, tests and limitations |
| `docs/SC02_v2_Credit_Optimization_Test.md` | v2 measurement and changed-score test |
| `docs/SC02_v3_Error_Recovery_Test_Log.md` | v3 configuration, eight test steps and recovery results |
| `workflows/SC02_v1_Dedupe.blueprint.json` | Original create/update prototype |
| `workflows/SC02_v2_Credit_Optimization.blueprint.json` | Score-based optimization |
| `workflows/SC02_v3_Error_Recovery.blueprint.json` | Latest prototype with manual recovery |
| `evidence/v1/`, `evidence/v2/`, `evidence/v3/` | Redacted screenshots linked from the documents |

## Reproduce the Tests

1. Import the chosen version into a separate Make scenario. Configure your own Google connection and a new webhook.
2. Select a fictional test spreadsheet in every Sheets module. v2/v3 use the tab `Credit_Test`; v1 uses `Submission_Master` in its original test setup.
3. Verify the five headers, field mappings, row-number mapping, filters and settings after import. Public resource placeholders are not executable configuration.
4. Run the corresponding document's test sequence. Verify which modules ran and check that the resulting sheet matches the expected record count and scores.
5. For v3, deliberately use `Credit_Test_Missing` in only the module being tested, resolve that saved execution manually, and restore the main scenario's configuration after each test. Keep IDs distinct between tests.

## Scope and Limitations

The implemented scope covers submission-ID lookup, conditional writes and manual recovery. Data normalization, independent input validation and human duplicate review are outside this prototype’s scope.

- No name/ID normalization, invalid-field checks or missing-field review is implemented.
- Search limit 1 does not identify duplicate IDs already in the sheet.
- Score-only optimization ignores other field changes if Score is unchanged.
- v3 tests one missing-tab error at three module locations, not three different error classes.
- Automatic retry, service outages, large datasets and concurrent writes are untested.
- Resuming Add after a successful write with a lost response can create a duplicate because the preceding Search is not repeated.
- Saved Update row numbers can become stale if other writers reorder rows.
- Sequential processing applies to this scenario only; unresolved executions can delay new requests.

## Security and Publication

Business test records are fictional. Public blueprints use disconnected resource references and spreadsheet placeholders; users must select their own resources. Check all exported metadata and screenshots for webhook URLs or tokens, connection labels, real email addresses, resource identifiers and embedded secrets before sharing.

Use redacted screenshots and sanitized blueprints for public sharing.

## Business Value

Under the tested conditions, repeated IDs do not add extra rows, unchanged scores avoid an unnecessary update, and configuration failures can be corrected using preserved execution data. Credit reduction is measured; time savings and production reliability are not established.
