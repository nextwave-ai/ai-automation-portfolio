# AI Lead Classifier

> Automatically score and qualify company leads using AI, with instant Telegram alerts for high-priority opportunities.

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-ff6d00)
![OpenAI](https://img.shields.io/badge/OpenAI-Chat_Model-10a37f)
![License](https://img.shields.io/badge/License-MIT-blue)
![Status](https://img.shields.io/badge/Status-Documented_Concept-yellow)

---

## Status

📄 **Documented workflow (screenshots included, no importable `workflow.json` in this version).** This workflow was built and executed in n8n — real output is visible in the included Google Sheets and Telegram screenshots — but the exported JSON is not currently included in this repository, so it cannot be imported directly. Verify node configuration against the screenshots below rather than assuming a one-click import.

---

## Overview

AI Lead Classifier is an n8n workflow that automatically scores and qualifies company leads added to a Google Sheet, using AI to assess strategic fit, and sends an instant Telegram alert for high-scoring leads so the sales team doesn't have to check the sheet manually.

---

## Business Problem

Sales teams often struggle with:

- Manual review of every new lead entered into a spreadsheet
- Inconsistent judgment on which companies are worth prioritizing
- Delayed awareness of high-value opportunities sitting unnoticed in a sheet
- No standardized scoring criteria across different people entering leads

---

## Solution

This workflow watches a Google Sheet for new or updated lead rows. Each lead (company, website, email, industry) is analyzed by an AI model that assigns a 0–10 score and a written justification, based on strategic fit signals. The result is written back into the sheet, and — for leads scoring above 7 — a formatted alert is sent instantly to a Telegram channel so the team never misses a high-quality lead.

---

## Key Features

- Google Sheets Trigger (fires on any row update)
- AI-Powered Company Scoring (0–10 scale) with a structured Output Parser
- Written justification (Reason) for every score
- Automatic Qualified/Not Qualified status
- Google Sheets logging (score, reason, status written back to the row)
- Telegram alert for high-scoring leads (score > 7)

---

## Workflow Architecture

Google Sheets Trigger (anyUpdate) → Basic LLM Chain (+ OpenAI Chat Model, + Output Parser) → Update Google Sheet (appendOrUpdate) → If Score > 7 → [true: Telegram Notification] / [false: no further action]

---

## Technology Stack

| Technology | Purpose |
|------------|---------|
| n8n | Workflow automation |
| OpenAI (via LangChain Chat Model) | Lead scoring and reasoning |
| Google Sheets API | Lead source and result storage |
| Telegram Bot API | High-priority lead alerts |

---

## Workflow Process

1. Google Sheets Trigger fires when a lead row is added or updated.
2. The Basic LLM Chain sends the company's Name, Website, Email, and Industry to the OpenAI Chat Model.
3. An Output Parser enforces a structured response: a Score (0–10) and a written Reason.
4. The sheet row is updated with Score, Reason, and a derived Status (e.g. Qualified).
5. If Score > 7, a Telegram Notification is sent to alert the sales team immediately.

---

## Example Input (sheet row)

| Company | Website | Email | Industry |
|---|---|---|---|
| OpenAI | openai.com | info@openai.com | AI |

## Example Output (written back to the row + Telegram alert)

**Sheet:**

| Score | Reason | Status |
|---|---|---|
| 10 | As a leading AI company that builds and deploys chatbots, agents, and platform integrations, OpenAI has an extremely high strategic interest in AI automation, workflow automation, integrations, and business process optimization for both products and internal operations. | Qualified |

**Telegram alert (score > 7):**

🔥 New High-Quality Lead

Company: OpenAI

Industry: AI

Score: 8

Reason: OpenAI, as a leading AI developer, has strong strategic interest in AI automation, workflows, integrations, chatbots, and process optimization, but its deep in-house capabilities and tendency to build bespoke solutions reduce the likelihood of purchasing off-the-shelf external solutions.

---

## Business Impact

Organizations using this workflow can:

- Score leads consistently instead of relying on manual judgment
- Get instant awareness of high-value leads via Telegram, without checking the sheet
- Reduce time-to-response for top prospects
- Keep a running, structured log of every lead's score and reasoning in one place

---

## Setup

This version does not include an importable workflow.json. To rebuild this workflow in n8n:

1. Create a Google Sheet with columns: Company, Website, Email, Industry, Score, Reason, Status.
2. Add a Google Sheets Trigger node (event: any update) pointed at that sheet.
3. Add a Basic LLM Chain node connected to an OpenAI Chat Model and an Output Parser, prompted to return a Score (0–10) and Reason based on the company's strategic fit for AI automation services.
4. Add a Google Sheets node (Update/Append or Update) to write Score, Reason, and a derived Status back to the matching row.
5. Add an IF node checking Score > 7.
6. On the true branch, add a Telegram node to send a formatted alert to your channel.

---

## Security

No credentials, tokens, or secrets are included in this repository. Configure your own OpenAI, Google Sheets, and Telegram credentials in n8n before running.

---

## Limitations

- No importable workflow.json in this version — rebuilding requires following the Setup steps above against the screenshots.
- Scoring criteria live entirely inside the AI prompt; there's no deterministic validation or allow-list on the Score/Status fields shown in the current screenshots (unlike other projects in this portfolio that validate every AI field).
- Trigger fires on any sheet update, not just new rows, so editing an existing row will re-trigger scoring.

---

## Future Improvements

- Export and include a proper workflow.json for direct import
- Add field-level validation on the AI's Score and Status output (matching the pattern used in this portfolio's other projects)
- CRM integrations (HubSpot, Salesforce)
- Lead enrichment APIs
- Restrict the trigger to new rows only, to avoid re-scoring on edits

---

## License

This project is licensed under the MIT License.

---

## Author

Built by **NextWave AI**
AI Automation Portfolio