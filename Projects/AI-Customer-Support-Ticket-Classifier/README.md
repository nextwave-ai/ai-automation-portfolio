# AI Customer Support Ticket Classifier

## Status

⚙️ **Runnable Workflow** — `workflow.json` is included in this folder and can be imported directly into n8n.

---

## Overview

AI Customer Support Ticket Classifier is an n8n workflow that receives an incoming customer support message via webhook, classifies it using OpenAI, and returns a structured classification — category, urgency, sentiment, a one-line summary, and a recommended next action — synchronously in the API response.

This is a **classification-only** service: it does not store tickets anywhere or notify anyone. It's designed to sit in front of an existing support system (helpdesk, CRM, or internal tool) and enrich each incoming ticket with structured metadata before that system takes over.

---

## Business Problem

Customer support teams receive many incoming messages that need to be read, categorized, and prioritized before anyone can act on them. Doing this manually is repetitive, slow, and inconsistent between different people.

---

## Solution

Every incoming message is sent to OpenAI, which returns a structured classification. A code node validates every field of the AI's response against an allow-list before returning it, so a malformed or unexpected AI response can never produce broken data for the caller.

---

## Key Features

- Webhook-based ticket intake
- AI-powered classification (category, urgency, sentiment)
- One-sentence AI summary of the core issue
- `needs_human` flag and a suggested next action
- Field-level validation with safe defaults on every field
- Synchronous JSON response — no external storage or notification step

---

## Workflow

1. Receive a support message via webhook (`POST /customer-support`).
2. Normalize the incoming name, email, and message fields.
3. Send the message to OpenAI (gpt-4.1-mini) for classification.
4. Parse the AI's JSON response and validate every field against an allow-list, with a full safe-fallback object if parsing fails entirely.
5. Return the structured classification as the webhook response.

---

## Technology Stack

- n8n (Webhook, Set, Code, Respond to Webhook)
- OpenAI (gpt-4.1-mini) via LangChain OpenAI node
- JSON / REST (webhook-based API)

---

## Workflow Architecture

Webhook (POST /customer-support) → Normalize Input → Classify with OpenAI → Parse Classification (validates every field, safe fallback) → Respond to Webhook

---

## AI Classification Logic

The OpenAI node returns:

- `category`: one of billing | technical | onboarding | account | general
- `urgency`: one of low | medium | high
- `sentiment`: one of positive | neutral | negative
- `summary`: one sentence describing the core issue
- `needs_human`: true/false
- `suggested_action`: one sentence describing the recommended next step

### Reliability: AI output is never trusted directly

The `Parse Classification` code node validates every field against an allow-list, falling back to a safe default for each one individually (e.g. an invalid `category` falls back to `general`). If the AI's response can't be parsed as JSON at all, the entire result falls back to a fully safe object with `needs_human: true` — ensuring a parsing failure routes the ticket to a human rather than silently returning broken or empty data.

---

## Example Input

```json
{
  "name": "John Smith",
  "email": "john@example.com",
  "message": "I forgot my password and cannot log in to my account."
}
```

## Example Output

```json
{
  "success": true,
  "classification": {
    "category": "account",
    "urgency": "medium",
    "sentiment": "neutral",
    "summary": "Customer is locked out of their account due to a forgotten password.",
    "needs_human": false,
    "suggested_action": "Send a password reset link to the customer's email."
  }
}
```

---

## Setup

1. Import `workflow.json` into n8n.
2. Add your OpenAI credentials to the **Classify with OpenAI** node.
3. Activate the workflow to expose the webhook.
4. Send a test request with the example input above to confirm the response.

No other credentials are required — this workflow does not integrate with Google Sheets, Slack, or any storage/notification system (see Limitations).

---

## Error Handling

- Every classification field is validated against an allow-list before use; invalid or missing values fall back to safe defaults instead of failing the request.
- If the AI's response can't be parsed as JSON at all, the workflow returns a fully safe fallback object with `needs_human: true`, so a broken AI response still results in the ticket being flagged for human review rather than silently failing.

---

## Security

No credentials, tokens, or secrets are included in this repository. Configure your own OpenAI credentials in n8n before running.

---

## Limitations

- This workflow does **not** log tickets to Google Sheets, a database, or any external system — it only classifies and returns the result. Storage is a natural next step, not yet implemented.
- No Slack, email, or other notification when `needs_human` is true — the caller is responsible for acting on that flag.
- No deduplication or ticket history — each request is classified independently with no memory of prior tickets from the same customer.

---

## Future Improvements

- Google Sheets or database logging of every classified ticket
- Slack/email alert when `needs_human` is true
- Gmail integration to classify tickets directly from an inbox
- CRM integration to attach classification to an existing customer record
- Dashboard for ticket volume and category trends

---

## Author

Built by **NextWave AI**
AI Automation Portfolio