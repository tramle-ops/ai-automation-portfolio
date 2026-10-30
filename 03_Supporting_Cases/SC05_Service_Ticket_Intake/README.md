# SC05 — Service Ticket Intake, Routing & Reprocessing

## Business problem

Service teams often receive requests with incomplete contact details, missing issue descriptions, or unsupported priority values. Without consistent routing, ownership and SLA deadlines are unclear. Repeating a follow-up email can also create a poor customer experience.

## Solution

This n8n solution validates and normalises a batch of service tickets, assigns operational queues and SLA deadlines, records tickets in a central register, and supports staff-led reprocessing after a correction. When a ticket lacks a subject but has a valid requester email, it sends one request-for-information email and records the delivery outcome.

The portfolio implementation uses a Manual Trigger and fictional structured sample data. It demonstrates ticket-processing logic; it is not connected to a live shared inbox.

## Workflow architecture

```text
SC05_Service_Ticket_Intake
  → normalise each ticket
  → SC05_Route_And_Register_Ticket

SC05_Reprocess
  → detects request_reprocess in Ticket_Register
  → prepares corrected ticket data
  → SC05_Route_And_Register_Ticket
  → records completion time
```

`SC05_Route_And_Register_Ticket` is the reusable routing workflow. Reusing it prevents the intake and reprocess flows from applying different routing rules.

## Key capabilities

- Normalises requester email, subject, and priority.
- Routes valid `high` tickets to `URGENT_QUEUE` with a 4-hour SLA.
- Routes valid `normal` tickets to `STANDARD_QUEUE` with a 24-hour SLA.
- Routes unsupported or blank priorities to `REVIEW_REQUIRED` with a 1-hour SLA.
- Creates an internal follow-up record when a ticket has no valid requester email.
- Sends a request-for-information email when the requester email is valid but the subject is missing.
- Records ticket state in `Ticket_Register` and email delivery state in `Email_Delivery_Log`.
- Lets staff correct a register record and request reprocessing with `request_reprocess`.
- Prevents a second request-for-information email when the delivery log for the same `ticket_id` is `SENT` or `SENDING`.

## Routing rules

| Condition | Queue / action | Status | SLA |
| --- | --- | --- | --- |
| Valid requester email and non-empty subject; priority = `high` | `URGENT_QUEUE` / `URGENT_SUPPORT` | `READY_FOR_DISPATCH` | 4 hours |
| Valid requester email and non-empty subject; priority = `normal` | `STANDARD_QUEUE` / `STANDARD_SUPPORT` | `READY_FOR_DISPATCH` | 24 hours |
| Valid requester email and non-empty subject; another or blank priority | `REVIEW_REQUIRED` / `TRIAGE_TEAM` | `PENDING_HUMAN_REVIEW` | 1 hour |
| Missing required information and no valid requester email | Internal follow-up / `INTAKE_TEAM` | `PENDING_CONTACT_REVIEW` | Not assigned |
| Missing subject and valid requester email | Requester email | `WAITING_FOR_CUSTOMER` | Not assigned |

Required information means both a non-empty requester email and a non-empty subject after normalisation. A sendable email must also match the workflow's email pattern.

## Reprocessing flow

1. A staff member corrects `requester_email`, `subject`, and/or `normalized_priority` in `Ticket_Register`.
2. The staff member adds a short `reprocess_note` and ticks `request_reprocess`.
3. `SC05_Reprocess` polls the register every five minutes.
4. It sets the ticket to `REPROCESSING`, records `reprocess_requested_at_utc`, clears the checkbox, and calls the reusable routing workflow.
5. After the child workflow succeeds, it records `last_reprocessed_at_utc`.

## Repository contents

```text
SC05_Service_Ticket_Intake/
├── README.md
├── workflows/
│   ├── SC05_Service_Ticket_Intake.json
│   ├── SC05_Route_And_Register_Ticket.json
│   └── SC05_Reprocess.json
├── docs/
│   ├── SC05_Test_Log.md
│   └── Runbook.md
├── data/
│   └── sample_tickets.json
└── evidence/
    ├── SC05-01_to_05_Initial_Routing_and_Delivery_Log.png
    ├── SC05-05_Request_Information_Email_T015.png
    ├── SC05-06_Reprocess_T005_Priority_Correction.png
    ├── SC05-07_Reprocess_T016_Email_Correction.png
    └── SC05-08_Reprocess_T016_Duplicate_Email_Prevention.png
```

## Setup

The public workflow files are templates: credentials, the source spreadsheet URL/ID, workflow IDs, and instance metadata have been removed. All workflows are inactive on import.

1. Create a Google Sheet with two tabs named `Ticket_Register` and `Email_Delivery_Log`, using the columns referenced by the Google Sheets mappings.
2. Import `SC05_Route_And_Register_Ticket.json` first, then `SC05_Service_Ticket_Intake.json`, and finally `SC05_Reprocess.json`.
3. In every Google Sheets or Google Sheets Trigger node, select your own credential and replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_URL` with the URL of your Sheet. Select the appropriate tab where prompted.
4. In the `Call SC05_Route_And_Register_Ticket` node and the `Reprocess via SC05_Route_And_Register_Ticket` node, select the imported `SC05_Route_And_Register_Ticket` workflow.
5. In the `SendRequestInformationEmail` node, select your own Gmail credential before testing customer follow-up.
6. Activate `SC05_Reprocess` only after checking its trigger sheet and five-minute polling interval. Keep the other workflows inactive until you are ready to test them.
7. Run the main workflow with the supplied fictional sample data.

## Test coverage

See [SC05_Test_Log.md](docs/SC05_Test_Log.md) for expected and actual results. Five focused screenshots provide evidence for the eight tests, including corrected-priority reprocessing, requester follow-up, and duplicate-email prevention.

## Limitations and operational considerations

- Email deduplication uses `ticket_id` only. Changing the recipient email for the same ticket does not create a new delivery key. A production design should use `ticket_id + delivery_type + recipient_email`.
- The workflow marks the delivery log as `SENDING` before sending email. It does not currently include an error branch that automatically marks a failed send as `FAILED`.
- `SC05_Reprocess` uses Google Sheets polling every five minutes. This is appropriate for the portfolio simulation, not high-volume or real-time processing.
- Do not apply checkbox validation to a large range of blank rows. It can generate unnecessary row-update events and cause new rows to be appended far below the active ticket list.
- The main workflow is a manual batch simulation. A production implementation would replace it with a controlled email, form, API, or webhook intake trigger.

## Project status

Portfolio simulation using fictional data. The tested routing, reprocessing, requester-email, and duplicate-email scenarios are documented in the test log.
