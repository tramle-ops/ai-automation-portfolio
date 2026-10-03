# Supporting Case 01 — Webhook Intake Logger

## Business Problem

An SME needs a simple way to receive submission data from another system and keep a searchable request log.

## Outcome

The workflow receives a JSON request, maps the payload fields, stores them in Google Sheets, and returns a confirmation response.

## Workflow

Postman → Make Custom Webhook → Google Sheets → Webhook Response

## Input

A fictional homework submission payload containing:

- submission_id
- assignment_id
- student_name
- class_name
- score
- valid

## Output

- One request log row in Google Sheets
- HTTP 200 response with a JSON confirmation message

## Test Summary

Three test cases were executed:

1. Valid payload
2. Number and boolean preservation
3. Missing student_name

## Known Limitation

The current version does not reject missing required fields. Validation will be added in a later iteration.

## Security Notes

- Only fictional data is used.
- The webhook URL is not published.
- No API key, password, access token, or Google credential is included.
- Screenshots hide the complete webhook URL.

## Tools

- Postman
- Make
- Google Sheets
