# AI CRM Automation

## Status

⚙️ **Runnable Workflow — tested end-to-end.** `Workflow.json` is included in this folder and can be imported directly into n8n. Verified with multiple live executions: correctly enriches new leads and correctly updates (not duplicates) existing ones on re-run.

---

## Overview

AI CRM Automation is an n8n workflow that automatically enriches CRM leads the moment they're added to a Google Sheet. Unlike a webhook-based intake system, this workflow is **passive and sheet-driven**: any team member (or another tool) can drop a new lead row into the sheet — no API call, no integration work required on the source side — and the workflow picks it up, analyzes it with AI, and writes the enrichment back into the same row.

---

## Business Problem

Sales and ops teams often collect leads in a spreadsheet from multiple sources — manual entry, CSV imports, other tools exporting into Sheets — before they're ready to build a full CRM integration.

This creates friction:

- Every new row needs manual research before it's useful (industry, sizing, likely intent)
- There's no consistent scoring or qualification across leads entered by different people
- Enrichment only happens when someone remembers to do it

---

## Solution

This workflow watches a Google Sheet for new rows. Whenever one is added, it normalizes the fields, sends the lead to OpenAI for analysis, validates the AI's response against a strict schema (rejecting any leftover template placeholders), and writes the enriched fields back into the same row — matched by email, so re-running the workflow updates the existing row instead of creating a duplicate.

---

## Key Features

- Google Sheets Trigger (polls for new rows every minute)
- Row Data Normalization
- AI Lead Enrichment (leadScore, qualification, summary, next step)
- Placeholder Detection & Rejection (rejects any leftover `[Your Name]`-style template text from the AI, falls back to a safe message instead)
- Field-Level Validation with Safe Defaults
- Match-by-Email Update Logic (prevents duplicate rows on re-run)

---

## Workflow

1. Google Sheets Trigger polls the sheet every minute for newly added rows.
2. Normalize the row's raw columns (Name, Email, Company, Industry, Notes) into clean variables.
3. Send the normalized lead to OpenAI (via a LangChain LLM Chain + Chat Model) for enrichment.
4. Parse the AI's JSON response and validate every field — reject and replace any value that still contains unfilled template brackets.
5. Write the enrichment fields back into the same row, matched by the lead's email address.

---

## Technology Stack

- n8n (Google Sheets Trigger, Set, LangChain LLM Chain, LangChain OpenAI Chat Model, Code, Google Sheets)
- OpenAI via LangChain integration
- Google Sheets API

---

## Workflow Architecture

```
Google Sheets Trigger (poll: every minute, event: row added)
        ↓
Normalize Row Data
        ↓
Analyze & Enrich Lead (LangChain LLM Chain)
        ↓ (model)
   OpenAI Chat Model
        ↓
Parse & Validate Enrichment (Code — rejects placeholders, applies safe defaults)
        ↓
Append or Update Row (matched by Email)
```

---

## AI Enrichment Logic

The LLM Chain analyzes each lead and returns:

- `leadScore`: integer 0–100
- `qualification`: one of `Qualified | Needs Nurturing | Not Qualified`
- `enrichmentSummary`: 2–3 sentences on the company and opportunity, using only the real data provided
- `recommendedNextStep`: one concrete action for the sales rep

The prompt explicitly instructs the model to never output template placeholders (e.g. `[Your Name]`, `[Company]`) and to write "Not provided" instead of inventing or leaving a field unfilled.

### Reliability: AI output is never trusted directly

The `Parse & Validate Enrichment` code node parses the AI's JSON and:
- Clamps `leadScore` to the 0–100 range, defaulting to `0` if unparseable
- Validates `qualification` against an allow-list, defaulting to `Needs Nurturing`
- Actively scans `enrichmentSummary` and `recommendedNextStep` for leftover template brackets (e.g. `[Your Name]`, `[Company]`) using a regex check, and replaces the entire field with a safe fallback message if any are found — this is a direct fix for an earlier version of this workflow that wrote unfilled template text straight into the CRM.

---

## Example Input (new sheet row)

| Name | Email | Company | Industry | Notes |
|---|---|---|---|---|
| Michael Chen | michael@retailplus.com | RetailPlus Inc | E-commerce | Small team, exploring options, no budget mentioned yet |

## Example Output (written back to the same row)

| leadScore | qualification | enrichmentSummary | recommendedNextStep |
|---|---|---|---|
| 50 | Needs Nurturing | RetailPlus Inc is a small e-commerce team currently exploring automation options with no confirmed budget yet. | Schedule a discovery call with Michael Chen to clarify needs, timeline, and decision-making process before proposing a plan. |

---

## Setup

1. Create a Google Sheet with headers: `Name, Email, Company, Industry, Notes, leadScore, qualification, enrichmentSummary, recommendedNextStep`
2. Import `Workflow.json` into n8n.
3. Connect your own Google Sheets credentials to both the **Google Sheets Trigger** and **Append or Update Row** nodes, and point them at your sheet.
4. Connect your own OpenAI credentials to the **OpenAI Chat Model** node.
5. Activate the workflow (or trigger manually via **Execute workflow** for testing).
6. Add a new row with lead data to test — the enrichment columns should populate automatically within a minute (or immediately on manual execution).

---

## Error Handling

- If the AI's JSON response is malformed or wrapped in markdown fences, it's stripped and re-parsed defensively; a fully unparseable response falls back to safe defaults across all fields.
- Any field containing leftover template placeholder text is detected and replaced, rather than being written to the sheet as-is.
- Matching on email (rather than row position) means re-running the workflow on the same lead updates the existing row instead of creating a duplicate.

---

## Security

No credentials, tokens, or secrets are included in this repository. Configure your own Google Sheets and OpenAI credentials in n8n before running.

---

## Limitations

- Trigger is poll-based (checks every minute), not instant — there's up to a ~60-second delay between a row being added and enrichment appearing.
- Matching is by email; leads without an email, or with a duplicate email across two different people, won't be handled correctly.
- No Slack or email notification — enrichment is written to the sheet only. (An earlier version of this README described Slack notifications; that integration does not exist in this workflow. See the separate **AI CRM Data Enrichment** project for a webhook-based workflow with Slack alerting.)
- No deduplication beyond the email match — if the same email appears in two different rows, only one will reliably get updated.

---

## Future Improvements

- Switch to a real-time trigger if/when the data source supports webhooks
- Add Slack or email alerting for high-scoring leads
- Deduplication across rows with the same email
- HubSpot / Salesforce / Airtable sync

---

## Author

Built by **NextWave AI**
AI Automation Portfolio