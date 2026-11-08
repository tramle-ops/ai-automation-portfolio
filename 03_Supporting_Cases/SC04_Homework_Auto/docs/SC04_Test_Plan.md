# SC04 Test Plan — Branch-Based Acceptance Test

## 1. Purpose and scope

This plan verifies the end-to-end behaviour of two connected workflows.

### Workflow 1 — `SC04_Intake_And_Match`

The workflow normalizes the submitted student name and class, then compares them with `Student_Master`. Each submission must reach exactly one outcome.

| Matching condition | Required outcome |
|---|---|
| **One exact active match** — exactly one active record has the same normalized name and class. | `AUTO_MATCHED`; the active student ID is saved. |
| **Multiple matches** — two or more active records have the same normalized name and class. | `PENDING_REVIEW` with `review_reason = MULTIPLE_MATCHES`. |
| **No exact match** — no record has the same normalized name and class. | `PENDING_REVIEW` with `review_reason = NO_EXACT_MATCH`. |
| **Inactive student** — one or more inactive records have the same normalized name and class, but no active exact match exists. | `PENDING_REVIEW` with `review_reason = INACTIVE_STUDENT`. |

An inactive record never prevents `AUTO_MATCHED` when there is exactly one active exact match.

### Workflow 2 — `SC04_Review`

This workflow processes a pending submission only after the coordinator completes the review fields and sets `review_ready = TRUE`. It must apply valid decisions, reject invalid input without unintended changes, and reset `review_ready` to `FALSE` after every attempt.

The plan also verifies that `Student_Report` and `Class_Report` use resolved matched submissions only.

## 2. Preconditions

1. `SC04_Intake_And_Match` and `SC04_Review` are published.
2. The Form is open with `HW-001` as the available assignment.
3. The workflow watches only rows where `review_ready = TRUE`.
4. `Student_Master`, `Submissions`, `Review_Log`, `Notification_Log`, `Student_Report`, and `Class_Report` are available.
5. Use a controlled email inbox for the resubmission test. Redact it, Sheet/Form URLs, and live student data from public evidence.
6. For every review test: populate the review fields, tick `review_ready`, wait for the execution to finish, then check the row and related log.

## 3. Test data conditions

Use the submitted names below, or equivalent names meeting the same condition. Do not depend on fixed student IDs except where an ID is deliberately selected in a review case.

| Condition | Example submitted name | Required `Student_Master` setup |
|---|---|---|
| Unique active | Nguyen An | Exactly one active exact candidate |
| Inactive student | Quoc Bao, Van Toan, Gia Hung, Le Chi, Pham Dung | No active and at least one inactive exact candidate |
| Multiple active | Mai Linh, Thanh Huong, Tran Binh | At least two active exact candidates |
| No exact match | Hoang Gian, Xuan Lann, An | No exact candidate |

For a standard-review confirmation, select an existing **active** student in the submitted class. For an inactive-review confirmation, select the original inactive student with the same normalized name and class.

## 4. Acceptance tests

Each row below identifies the tested workflow branch and its acceptance criteria. For review tests, populate the review fields, set `review_ready = TRUE`, wait for the execution to finish, then verify the results.

| ID | Workflow and branch | Action | Expected result |
|---|---|---|---|
| **SC04-01** | Intake — all four matching conditions | Submit a batch that covers all four conditions in section 1. | Every row is processed independently. One exact active match becomes `AUTO_MATCHED` with the active ID; multiple matches becomes `PENDING_REVIEW` / `MULTIPLE_MATCHES`; no exact match becomes `PENDING_REVIEW` / `NO_EXACT_MATCH`; inactive student becomes `PENDING_REVIEW` / `INACTIVE_STUDENT`. No review log or notification is created. |
| **SC04-02** | Review — standard confirmation | On a fresh `MULTIPLE_MATCHES` row, choose `CONFIRMED`, select a valid active student in the submitted class, and set `review_ready = TRUE`. Example: Mai Linh → `STU-005`. | `match_status = MANUALLY_MATCHED`; selected ID, `reviewed_by`, and `reviewed_at` are saved; `review_ready = FALSE`; one `Review_Log` row exists. |
| **SC04-03** | Review — invalid standard selected ID | On a fresh `MULTIPLE_MATCHES` or `NO_EXACT_MATCH` row, choose `CONFIRMED` and select an ID that is missing, inactive, or in another class. Example: Thanh Huong → `STU-014`. | `review_validation_error = Selected student must exist, be active, and be in the submitted class`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; no log or email. |
| **SC04-04** | Review — missing standard selected ID | On a fresh standard-review row, choose `CONFIRMED`, leave `matched_student_id` blank, and set `review_ready = TRUE`. Example: Tran Binh. | `review_validation_error = Select a student ID before confirming`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; no log or email. |
| **SC04-05** | Review — resubmission required | On a fresh `NO_EXACT_MATCH` or `MULTIPLE_MATCHES` row, choose `NOT_CONFIRMED` and set `review_ready = TRUE`. Example: Hoang Gian. | `match_status = RESUBMISSION_REQUIRED`; no matched ID; reviewer fields are saved; one `Review_Log` row exists; one `Notification_Log` row has `email_send_status = SENT`, timestamp, and message ID; exactly one email reaches the controlled inbox. |
| **SC04-06** | Review — invalid standard result | On a fresh standard-review row, choose `REJECTED` and set `review_ready = TRUE`. Example: Xuan Lann. | `review_validation_error = coordinator_review_result should be confirmed or not_confirmed`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; no log or email. |
| **SC04-07** | Review — inactive student confirmed | On a fresh `INACTIVE_STUDENT` row, choose `CONFIRMED`, select the matching inactive ID, and set `review_ready = TRUE`. Example: Quoc Bao → `STU-010`. | The selected `Student_Master.active` becomes `TRUE`; submission becomes `MANUALLY_MATCHED`; reviewer fields are saved; one `Review_Log` row exists. |
| **SC04-08** | Review — invalid inactive selected ID | On a fresh `INACTIVE_STUDENT` row, choose `CONFIRMED` and select an ID with a different name or class. Example: Van Toan → `STU-001`. | `review_validation_error = Selected student must exist and match the submitted name and class.`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; selected student is unchanged; no log or email. |
| **SC04-09** | Review — missing inactive selected ID | On a fresh `INACTIVE_STUDENT` row, choose `CONFIRMED`, leave `matched_student_id` blank, and set `review_ready = TRUE`. Example: Gia Hung. | `review_validation_error = Select a student ID before confirming`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; no Student Master update, log, or email. |
| **SC04-10** | Review — inactive student rejected | On a fresh `INACTIVE_STUDENT` row, choose `REJECTED` and set `review_ready = TRUE`. Example: Le Chi. | `match_status = REJECTED`; matched ID is blank; the inactive `Student_Master` row remains inactive; reviewer fields and one `Review_Log` row are created; no notification is sent. |
| **SC04-11** | Review — invalid inactive result | On a fresh `INACTIVE_STUDENT` row, choose `NOT_CONFIRMED` and set `review_ready = TRUE`. Example: Pham Dung. | `review_validation_error = coordinator_review_result should be confirmed or rejected`; status remains `PENDING_REVIEW`; `review_ready = FALSE`; no log or email. |
| **SC04-12** | Review — invalid review state | On a pending `NO_EXACT_MATCH` row, leave `coordinator_review_result = PENDING` and set `review_ready = TRUE`. | `review_validation_error = Check match_status, review_reason, coordinator_review_result and reviewed_at`; `review_ready = FALSE`; no status resolution, Student Master update, log, or email. |
| **SC04-13** | Reporting — student report | Compare `Student_Report` with final `Submissions` rows. | One row exists for each resolved `assignment_id` + `student_id` + `class_code`. `attempt_count`, `highest_score`, and `official_name` match the resolved submissions and `Student_Master`. `PENDING_REVIEW`, `REJECTED`, and `RESUBMISSION_REQUIRED` rows are excluded. |
| **SC04-14** | Reporting — class report | Compare `Class_Report` with `Student_Master` and `Student_Report`. | `submitted_student_count` equals resolved students in `Student_Report`; `active_student_count` equals active students in `Student_Master`; `completion_rate = submitted_student_count / active_student_count`; average and highest score use resolved student results only. |

## 5. Evidence

Keep one public-safe screenshot per tested outcome, with the node result or Sheets data that proves the final effect.

| Evidence | Test(s) | Evidence file |
|---|---|---|
| Intake outcome grid | SC04-01 | `T01_intake.png` |
| Standard review: manual match | SC04-02 | `T02_manual_match.png` |
| Invalid standard selected student | SC04-03 | `T03_invalid_std_student.png` |
| Missing standard student ID | SC04-04 | `T04_missing_std_id.png` |
| Resubmission and sent email | SC04-05 | `T05_resubmit.png`; `T05_email_sent.png` |
| Invalid standard result | SC04-06 | `T06_invalid_std_result.png` |
| Inactive confirmation and reactivation | SC04-07 | `T07_reactivate.png` |
| Invalid inactive selected student | SC04-08 | `T08_invalid_inactive_student.png` |
| Missing inactive student ID | SC04-09 | `T09_missing_inactive_id.png` |
| Inactive review rejected | SC04-10 | `T10_rejected.png` |
| Invalid inactive result | SC04-11 | `T11_invalid_inactive_result.png` |
| Invalid review state | SC04-12 | `T12_invalid_review_state.png` |
| Student-level reporting | SC04-13 | `T13_student_report.png` — show the relevant `Submissions` rows and `Student_Report` together. |
| Class-level reporting | SC04-14 | `T14_class_report.png` — show the relevant `Student_Master` rows and `Class_Report` together. |

Evidence in this package has been redacted to remove personal email addresses, Sheet/Form URLs and IDs, browser profiles, and other account metadata. The student records are fictional portfolio data. Full-canvas workflow screenshots are supporting context, not primary acceptance evidence.

## 6. Pass criteria

SC04 passes when all 14 cases meet their expected result, each validation branch resets `review_ready` and causes no side effect, manual matching happens only after a valid coordinator decision, each resubmission produces one notification record and one sent email, and reports count only resolved matched submissions.
