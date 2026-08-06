# AI Proposal Generator

## Overview

AI Proposal Generator is an n8n automation that watches a Google Sheet for new incoming job leads and automatically drafts a ready-to-send freelance proposal message for each one using OpenAI.

When a new row is added with job details (title, description, client needs, suggested price, delivery time), the workflow generates a personalized proposal message and writes it back into the same row — no manual copy-pasting or proposal writing required.

---

## Status

⚙️ Runnable Workflow — tested end-to-end in n8n with a live Google Sheet and OpenAI credentials. All nodes execute successfully on new row addition.

---

## Business Problem

Freelancers and agencies fielding multiple leads spend significant time writing a first-draft proposal message for every inquiry:

- Repetitive, similar messages written from scratch each time
- Slower response time to new leads
- Inconsistent tone or missing details across proposals
- Manual tracking of what's been quoted to whom

---

## Solution

This workflow turns a Google Sheet into a lightweight lead-to-proposal pipeline. As soon as a new lead row is added, AI drafts a complete, personalized proposal message referencing the client's specific job details, price, and timeline — written back into the sheet, ready to review and send.

---

## Workflow Architecture

Google Sheets Trigger (Row Added) → Basic LLM Chain (OpenAI) → Validate Proposal (Code) → Append or Update Row in Sheet [matched on: JobTitle]

**Trigger:** Polls the sheet and fires only on newly added rows (not on every edit), preventing re-trigger loops when the workflow writes its own output back to the row.

**Generation:** An LLM Chain node, backed by an OpenAI Chat Model, reads the row's JobTitle, JobDescription, ClientNeeds, SuggestedPrice, and DeliveryTime, and generates a first-person proposal message in a professional freelance tone.

**Validation:** A Code node checks the generated text before it's written anywhere — rejecting empty/too-short output and any bracket-style placeholder text (e.g. `[Client Name]`), so nothing unfinished ever reaches the sheet.

**Write-back:** The validated proposal is written into the same row via Append or Update Row, matched on the JobTitle column — so re-running the workflow updates the existing row instead of creating a duplicate.

---

## Key Features

- Automatic proposal drafting from structured lead data
- Row-added trigger (avoids re-trigger loops on self-updates)
- Placeholder-rejection validation layer before write-back
- Row matching to prevent duplicate entries
- Single source of truth (one sheet holds leads + generated proposals)

---

## Technology Stack

- n8n
- Google Sheets (Trigger + Read/Write)
- OpenAI API (gpt-5-mini)
- LangChain LLM Chain node
- JavaScript (Code node validation)

---

## Example Input (New Sheet Row)

| JobTitle | JobDescription | ClientNeeds | SuggestedPrice | DeliveryTime |
|---|---|---|---|---|
| Build an n8n workflow | Need an automation that reads leads from Google Sheets and sends Telegram notifications | Basic error handling, configurable trigger, deduplication | $350 | 3 business days |

---

## Example Output (Written Back to Same Row)

> Hello — I can build a reliable n8n workflow that reads leads from your Google Sheets and sends Telegram notifications for new entries. My approach: 1) Configure a Google Sheets trigger to poll or push new rows, 2) Add basic error handling and logging, 3) Add deduplication logic to avoid duplicate alerts, 4) Send formatted Telegram notifications. I can deliver this within 3 business days for $350. Do you have a preference for polling vs. a push-based trigger for detecting new leads?

---

## Reliability

- **Placeholder prevention:** the Validate Proposal node rejects any output containing bracketed placeholder text before it's ever written to the sheet.
- **No duplicate rows:** Append/Update matches on JobTitle, so reruns update the existing lead instead of creating a new one.
- **No infinite loop:** trigger is set to Row Added (not Any Update), so the workflow's own write-back doesn't re-fire itself.

---

## Business Value

- Cuts first-draft proposal time from minutes to seconds per lead
- Keeps tone and structure consistent across every response
- Centralizes lead + proposal tracking in a single sheet
- Frees up time to focus on qualified leads instead of repetitive writing

---

## Future Improvements

- Email delivery of the generated proposal (Gmail integration)
- PDF/Google Docs export for formal proposal documents
- Slack/Telegram notification when a new proposal is ready for review
- Multi-language proposal generation

---

## Author

Built by **NextWave AI**

AI Automation Portfolio