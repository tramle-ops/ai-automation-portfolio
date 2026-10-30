# SC05 Test Log

## Test environment

- **Workflows:** `SC05_Service_Ticket_Intake`, `SC05_Route_And_Register_Ticket`, and `SC05_Reprocess`
- **Data stores:** `Ticket_Register` and `Email_Delivery_Log` in Google Sheets
- **Email channel:** Gmail test mailbox
- **Test data:** Fictional ticket records only

## Results

| Test ID | Scenario | Expected result | Actual result | Status |
| --- | --- | --- | --- | --- |
| SC05-01 | Tickets have a valid `normalized_email`, valid `normalized_subject`, and valid `high` `normalized_priority`: T-001; T-012 after trim and case normalization. | Route to `URGENT_QUEUE`, set `READY_FOR_DISPATCH`, and assign a 4-hour SLA. | T-001 and T-012 were routed to `URGENT_QUEUE` with `READY_FOR_DISPATCH` and a 4-hour SLA. | PASS |
| SC05-02 | Ticket T-002 has a valid `normalized_email`, valid `normalized_subject`, and valid `normal` `normalized_priority`. | Route to `STANDARD_QUEUE`, set `READY_FOR_DISPATCH`, and assign a 24-hour SLA. | T-002 was routed to `STANDARD_QUEUE` with `READY_FOR_DISPATCH` and a 24-hour SLA. | PASS |
| SC05-03 | Ticket T-005 has valid required information but unsupported `critical` `normalized_priority`. | Route to `REVIEW_REQUIRED`, set `PENDING_HUMAN_REVIEW`, and assign a 1-hour SLA. | T-005 was routed to `REVIEW_REQUIRED` with `PENDING_HUMAN_REVIEW` and a 1-hour SLA. | PASS |
| SC05-04 | Tickets have a missing or invalid `normalized_email` despite a usable subject: T-003 (empty), T-007 (spaces only), T-008/T-009 (requester or email missing), T-010 (`null`), and T-011 (number). | Create an internal follow-up record; do not send requester email. | T-003, T-007, T-008, T-009, T-010, and T-011 were recorded as `PENDING_CONTACT_REVIEW` with internal follow-up; no requester email was sent. | PASS |
| SC05-05 | Tickets have a valid `normalized_email` but a missing or invalid `normalized_subject`: T-004 (empty), T-013 (spaces only), T-014 (`null`), and T-015 (number). | Send one request-for-information email, record delivery, and set `WAITING_FOR_CUSTOMER`. | T-004, T-013, T-014, and T-015 each sent one requester email, created one `SENT` delivery-log record, and moved to `WAITING_FOR_CUSTOMER`. | PASS |
| SC05-06 | Reprocess T-005 after staff corrects `normalized_priority` from `critical` to `high`. | Re-route the ticket using the corrected priority and recalculate the SLA. | T-005 moved from `REVIEW_REQUIRED` to `URGENT_QUEUE` with `READY_FOR_DISPATCH` and a 4-hour SLA. Reprocess timestamps were recorded and the checkbox was cleared. | PASS |
| SC05-07 | Reprocess T-016 after staff adds a valid requester email while `normalized_subject` remains invalid. | Send one request-for-information email, record the delivery, and set `WAITING_FOR_CUSTOMER`. | After its email was corrected, T-016 sent one requester email, created one `SENT` delivery-log record, and moved to `WAITING_FOR_CUSTOMER`. | PASS |
| SC05-08 | Reprocess unchanged T-016 after its requester email has already been sent. | Do not send a duplicate requester email. | T-016 remained in `WAITING_FOR_CUSTOMER`; its delivery log still had one `SENT` record and no second requester email was created. | PASS |

## Evidence mapping

The eight tests are evidenced by five focused screenshots. This keeps the portfolio concise while retaining traceability: the initial-routing screenshot shows the full batch outcome, the email screenshot confirms the customer-facing result, and each reprocess screenshot shows its own register and delivery-log state.

Browser address bars and account avatars have been redacted in the public evidence copies. The redactions do not cover any test result.

| Test ID | Evidence file | Verified in evidence |
| --- | --- | --- |
| SC05-01 | `SC05-01_to_05_Initial_Routing_and_Delivery_Log.png` | T-001 and T-012 show `URGENT_QUEUE`, `READY_FOR_DISPATCH`, and a 4-hour SLA. |
| SC05-02 | `SC05-01_to_05_Initial_Routing_and_Delivery_Log.png` | T-002 shows `STANDARD_QUEUE`, `READY_FOR_DISPATCH`, and a 24-hour SLA. |
| SC05-03 | `SC05-01_to_05_Initial_Routing_and_Delivery_Log.png` | T-005 shows `REVIEW_REQUIRED`, `PENDING_HUMAN_REVIEW`, and a 1-hour SLA. |
| SC05-04 | `SC05-01_to_05_Initial_Routing_and_Delivery_Log.png` | T-003, T-007, T-008, T-009, T-010, and T-011 show `PENDING_CONTACT_REVIEW` with internal follow-up. |
| SC05-05 | `SC05-01_to_05_Initial_Routing_and_Delivery_Log.png` and `SC05-05_Request_Information_Email_T015.png` | T-004, T-013, T-014, and T-015 show `WAITING_FOR_CUSTOMER` and `SENT` delivery-log records; the Gmail message for T-015 confirms the customer-facing email. |
| SC05-06 | `SC05-06_Reprocess_T005_Priority_Correction.png` | T-005 shows the corrected `high` priority, `URGENT_QUEUE`, `READY_FOR_DISPATCH`, a 4-hour SLA, reprocess timestamps, and a cleared checkbox. |
| SC05-07 | `SC05-07_Reprocess_T016_Email_Correction.png` | T-016 shows `WAITING_FOR_CUSTOMER` and exactly one `SENT` record after a valid requester email is added. |
| SC05-08 | `SC05-08_Reprocess_T016_Duplicate_Email_Prevention.png` | The unchanged reprocess of T-016 leaves exactly one `SENT` delivery-log record, proving that no duplicate email was created. |

## Notes

- Email de-duplication currently uses `ticket_id` as the delivery key.
- The workflow prevents another send when an existing delivery-log record is `SENT` or `SENDING`.
- A production implementation should add automated failure handling to change a failed `SENDING` record to `FAILED`.
