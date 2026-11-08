# Workflow Charter — Homework Submission Matching and Reporting

## Business problem

Students may submit homework using inconsistent names. The coordinator then has to identify students manually before reporting can be trusted, delaying results and creating a risk of inaccurate attempt counts, scores, and completion rates.

## Outcome

Within the Google Sheets polling interval, each new form response is preserved and either linked to one active student or routed to review. Confirmed submissions feed student- and class-level reports without deleting earlier attempts.

## Users and systems

| Area | Detail |
|---|---|
| Primary user | Education Coordinator — maintains reference data and resolves uncertain matches. |
| Other users | Students submit homework; teachers use the reports. |
| Automation | n8n runs the intake/match and coordinator-review workflows. |
| Data store | Google Forms captures responses; Google Sheets stores records, logs, and reports. |
| Notification | Gmail sends the resubmission request after a valid `NOT_CONFIRMED` review. |

## Inputs and outputs

**Input:** timestamp, student email, assignment ID, entered name, class code, and score from `Form_Responses`; `Student_Master` is the reference table.

**Outputs:** a processed `Submissions` record, `Review_Log`, optional `Notification_Log` entry and resubmission email, plus `Student_Report` and `Class_Report`.

## Matching policy

| Condition | Result |
|---|---|
| Exactly one active record matches normalized name and class | `AUTO_MATCHED` |
| Two or more active records match | `PENDING_REVIEW` / `MULTIPLE_MATCHES` |
| No exact record matches | `PENDING_REVIEW` / `NO_EXACT_MATCH` |
| Inactive record(s) match and no active record matches | `PENDING_REVIEW` / `INACTIVE_STUDENT` |

An inactive record never blocks automatic matching when exactly one active exact match exists.

## Success measures

- Each received form response produces one processed submission record.
- Uncertain cases are never automatically approved.
- Review validation failures reset `review_ready` and produce no unintended update or email.
- Resubmission email delivery is logged and checked before another email is sent.
- Reports include only `AUTO_MATCHED` and `MANUALLY_MATCHED` submissions.

## Scope boundaries

The workflow does not infer identity with fuzzy matching, change a score, contact parents, or replace an audit trail. All portfolio data and evidence must remain fictional and public-safe.
