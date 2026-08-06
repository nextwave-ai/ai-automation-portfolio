# AI Lead Classifier

## Status

⚙️ **Runnable Workflow — tested end-to-end.** `Workflow.json` is included in this folder and can be imported directly into n8n. Verified with multiple live executions across low, medium, and high-scoring leads, including confirmed Google Sheets logging.

---

## Overview

AI Lead Classifier is an n8n workflow that automatically scores and qualifies company leads added to a Google Sheet, using AI to assess strategic fit, and sends an instant Telegram alert for high-scoring leads so the sales team doesn't have to check the sheet manually.

---

## Business Problem

Sales teams often struggle with manual review of every new lead entered into a spreadsheet, inconsistent judgment on which companies are worth prioritizing, delayed awareness of high-value opportunities sitting unnoticed in a sheet, and no standardized scoring criteria across different people entering leads.

---

## Solution

This workflow watches a Google Sheet for new or updated lead rows. Each lead (company, website, email, industry) is analyzed by an AI model that assigns a 0-10 score and a written justification, based on strategic fit signals. A code node validates and clamps the score, applies a safe fallback reason if the AI response is missing or too short, and derives a Qualified/Needs Review/Not Qualified status deterministically from the numeric score rather than trusting an AI-generated label. The result is written back into the sheet, and for leads scoring above 7, a formatted alert is sent instantly to a Telegram channel.

---

## Key Features

- Google Sheets Trigger (fires on any row added or updated)
- AI-powered company scoring (0-10 scale) with a written justification
- Defensive parsing that handles both structured Output Parser results and raw JSON-in-a-string model output
- Score clamped to the 0-10 range with a safe fallback if the AI response is missing or malformed
- Status (Qualified / Needs Review / Not Qualified) derived deterministically from the numeric score, never trusted from the AI directly
- Google Sheets logging (score, reason, status written back to the row)
- Telegram alert for high-scoring leads (score greater than 7)

---

## Workflow

1. Google Sheets Trigger fires when a lead row is added or updated.
2. The Basic LLM Chain sends the company's Name, Website, Email, and Industry to the OpenAI Chat Model, asking for a score and a written reason.
3. Validate Score parses the AI's response, clamps the score to 0-10, falls back to a safe reason if needed, and derives Status from the score.
4. Update Google Sheet writes Score, Reason, and Status back into the matching row, identified by Company name.
5. If Score is greater than 7, a Telegram Notification is sent to alert the sales team immediately.

---

## Technology Stack

- n8n (Google Sheets Trigger, LangChain LLM Chain, LangChain OpenAI Chat Model, Output Parser, Code, Google Sheets, IF, Telegram)
- OpenAI (gpt-4.1-mini) via LangChain integration
- Google Sheets API
- Telegram Bot API

---

## Workflow Architecture

Google Sheets Trigger (event: row added or updated) → Basic LLM Chain (+ OpenAI Chat Model, + Output Parser) → Validate Score (Code — parses AI output defensively, clamps score, derives status) → Update Google Sheet (matched by Company) → If Score greater than 7 → true: Telegram Notification / false: no further action

---

## AI Scoring Logic

The AI is asked to return a `score` (0-10) and a `reason` (2-3 sentences) evaluating a company's strategic fit as a buyer of AI automation services.

Scoring guide used in the prompt: 8-10 for strong explicit interest in automation or integrations where the company profile clearly suggests they would buy external services; 5-7 for a plausible fit reduced by factors like deep in-house technical capability; 0-4 for weak or no fit.

### Reliability: AI output is never trusted directly

The `Validate Score` code node handles two different shapes of model output depending on whether the Output Parser is active: a structured object, or a raw string containing JSON (which is stripped of markdown fences and parsed). The numeric score is clamped to 0-10 with a default of 0 if unparseable, the reason falls back to a safe generic message if missing or too short, and — critically — `Status` (Qualified / Needs Review / Not Qualified) is computed from the numeric score in code, never taken directly from the AI's own labeling. This is what guarantees the Telegram alert threshold (`Score > 7`) is always evaluated against a clean, validated number.

---

## Example Input (new sheet row)

| Company | Website | Email | Industry |
|---|---|---|---|
| Acme Corp | acme.com | info@acme.com | Software |

## Example Output (written back to the same row)

| Score | Reason | Status |
|---|---|---|
| 6 | Acme Corp shows minimal public information about automation needs, making strategic fit plausible but unconfirmed without further discovery. | Needs Review |

## Example Telegram Alert (score greater than 7)

Sent only when a lead's Score exceeds 7, formatted as:

🔥 New High-Quality Lead — Company, Industry, Score, and Reason.

---

## Setup

1. Create a Google Sheet with headers: Company, Website, Email, Industry, Score, Reason, Status.
2. Import `Workflow.json` into n8n.
3. Connect your own Google Sheets credentials to both the Google Sheets Trigger and Update Google Sheet nodes, and point them at your sheet.
4. Connect your own OpenAI credentials to the OpenAI Chat Model node.
5. Connect your own Telegram Bot credentials to the Telegram Notification node, and set your channel/chat ID.
6. Activate the workflow (or trigger manually via Execute workflow for testing).
7. Add or edit a row with lead data to test — Score, Reason, and Status should populate automatically.

---

## Error Handling

- The AI's response is parsed defensively regardless of whether it arrives as a structured object or a JSON string wrapped in markdown fences.
- An unparseable or missing score defaults to 0 rather than breaking the workflow.
- An unparseable or missing reason falls back to a safe generic message rather than being left blank.
- Status is always derived from the validated numeric score, so it can never disagree with the number shown in the same row.

---

## Security

No credentials, tokens, or secrets are included in this repository. Configure your own Google Sheets, OpenAI, and Telegram credentials in n8n before running.

---

## Limitations

- Trigger fires on any sheet update, not just new rows, so editing an existing row will re-trigger scoring.
- Polling-based trigger checks every minute rather than firing instantly.
- No deduplication beyond matching by Company name; two different leads with the same company name will collide.
- Scoring reflects the AI's judgment of general strategic fit for AI automation services specifically, not a generic lead-quality score.

---

## Future Improvements

- Restrict the trigger to new rows only, to avoid re-scoring on edits
- CRM integrations (HubSpot, Salesforce)
- Lead enrichment APIs for richer company data
- Deduplication by a unique identifier (e.g. domain) rather than company name

---

## Author

Built by **NextWave AI**
AI Automation Portfolio