# SC05 Runbook

## Purpose

Use this runbook to operate and recover the Service Ticket Intake, Routing & Reprocessing workflow. The system has three workflows:

- `SC05_Service_Ticket_Intake`: receives a manual batch of fictional test tickets, normalises each item, and calls the routing workflow.
- `SC05_Route_And_Register_Ticket`: applies routing, SLA, register, and requester-email rules.
- `SC05_Reprocess`: monitors staff-requested reprocessing in `Ticket_Register`.

## First-time configuration

The public workflow exports do not include credentials, source spreadsheet details, workflow IDs, or instance metadata. Import the routing workflow first, then select it again in each parent Execute Workflow node. In every Google Sheets or Google Sheets Trigger node, select your own credential, provide your own spreadsheet URL, and select either `Ticket_Register` or `Email_Delivery_Log` as appropriate. Select your Gmail credential in `SendRequestInformationEmail` before testing the requester-email route.

## Normal operation

1. Run `SC05_Service_Ticket_Intake` with a controlled batch of tickets.
2. Check `Ticket_Register` for each ticket's `queue`, `owner_team`, `processing_status`, and `sla_due_at_utc`.
3. Check `Email_Delivery_Log` when a ticket has a valid requester email but a missing subject.
4. Use the queue and SLA deadline to assign human follow-up.

Expected outcomes:

| Ticket state | Operational result |
| --- | --- |
| Valid High ticket | `URGENT_QUEUE`, 4-hour SLA, `READY_FOR_DISPATCH` |
| Valid Normal ticket | `STANDARD_QUEUE`, 24-hour SLA, `READY_FOR_DISPATCH` |
| Unsupported priority | `REVIEW_REQUIRED`, 1-hour SLA, `PENDING_HUMAN_REVIEW` |
| No valid requester email | Internal follow-up, `PENDING_CONTACT_REVIEW` |
| Valid requester email but missing subject | One requester email, `WAITING_FOR_CUSTOMER` |

## Reprocess a corrected ticket

1. Open `Ticket_Register` and locate the ticket by `ticket_id`.
2. Correct the relevant field or fields:
   - `requester_email` when a valid contact address is available;
   - `subject` when the requester has supplied a usable issue description;
   - `normalized_priority` when triage confirms `high` or `normal`.
3. Add a concise explanation in `reprocess_note`.
4. Tick `request_reprocess`.
5. Wait for the active `SC05_Reprocess` workflow to poll the sheet (up to five minutes).
6. Confirm that the checkbox is cleared and `reprocess_requested_at_utc` and `last_reprocessed_at_utc` are populated.
7. Confirm that the resulting queue, status, SLA, and delivery log match the corrected information.

Do not tick `request_reprocess` repeatedly while the ticket is already being processed.

## Verify a requester email

When a ticket has a valid email but a missing subject:

1. Confirm `Ticket_Register` shows `WAITING_FOR_CUSTOMER` and `contact_route = REQUESTER_EMAIL`.
2. Check `Email_Delivery_Log` for the same `ticket_id`.
3. A successful delivery must show `email_send_status = SENT`, `email_sent_at`, and `email_message_id`.
4. Do not infer that an email was sent merely because the ticket is in `WAITING_FOR_CUSTOMER`; use the delivery log as the source of truth.

## Verify duplicate-email prevention

1. Confirm the ticket has one `Email_Delivery_Log` record with `email_send_status = SENT`.
2. Request reprocessing without changing the ticket's missing-subject state.
3. Confirm that no additional requester email is sent and the delivery log remains one record for that `ticket_id`.

## Recovery

### A ticket does not reprocess

1. Confirm that `SC05_Reprocess` is active and its Google Sheets Trigger points to `Ticket_Register`.
2. Confirm that `request_reprocess` is checked and the ticket has a non-empty `ticket_id`.
3. Check the most recent workflow execution. Identify the first failed or unexecuted node.
4. Correct the underlying data or configuration issue before requesting reprocessing again.
5. After success, verify timestamps and the final register state.

### A ticket remains `REPROCESSING`

1. Open the relevant `SC05_Reprocess` execution.
2. Check whether `SC05_Route_And_Register_Ticket` completed successfully.
3. If it failed, correct the issue first; then tick `request_reprocess` again.
4. If the child workflow succeeded but the final timestamp was not written, rerun only after confirming the ticket's register record is correct.

### An email is stuck at `SENDING`

The current workflow marks `SENDING` before Gmail sends the email, but it does not automatically record a failed send.

1. Check the Gmail node execution and the Sent folder to establish whether the email was actually delivered.
2. If the email was sent, update the delivery log to `SENT` and record the message ID if available. Do not resend.
3. If the email was not sent, update `email_send_status` to `FAILED`, record a brief reason in `last_error`, correct the root cause, then request reprocessing.
4. Do not leave an undelivered email as `SENDING`; the dedupe check will block a new attempt.

### Google Sheets polling creates too many events or new rows appear far below the active data

1. Remove checkbox data validation from unused blank rows.
2. Keep the `request_reprocess` checkbox only on rows containing real tickets.
3. Wait for Google API quota limits to clear before repeatedly testing or refreshing nodes.
4. For a high-volume production solution, replace sheet polling with an event-driven mechanism such as a webhook or controlled script trigger.

## Data-handling rules

- Use fictional data in all public portfolio screenshots, samples, and workflow exports.
- Do not add real customer email addresses, ticket details, credentials, or active webhook URLs to the repository.
- Confirm that exported workflows contain no secrets before uploading.
- Public evidence must not show a browser address bar, account avatar, profile name, or other personal identity data. This package uses opaque redaction where needed.
