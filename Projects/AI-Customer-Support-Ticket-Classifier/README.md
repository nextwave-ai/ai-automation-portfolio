# AI Customer Support Ticket Classifier

## Overview

AI Customer Support Ticket Classifier is an n8n workflow that automatically analyzes incoming customer support requests using OpenAI.

The workflow classifies each ticket, determines its priority, generates an AI summary, and stores the structured result for the support team.

This automation reduces manual triage time and helps support teams respond faster to customer requests.

---

## Business Problem

Customer support teams receive dozens or hundreds of tickets every day.

Manually reading every message, assigning categories, and determining urgency is repetitive, slow, and inconsistent.

This results in:

- Slow response times
- Incorrect ticket routing
- High operational costs
- Inconsistent prioritization

---

## Solution

This workflow automates the first stage of customer support.

Every incoming support request is processed by OpenAI, classified into predefined categories, prioritized, summarized, and logged automatically.

The support team immediately receives structured information instead of raw customer messages.

---

## Key Features

- AI Ticket Classification
- Priority Detection
- AI Ticket Summary
- Structured JSON Output
- Google Sheets Integration
- Automated Ticket Logging
- n8n Workflow Automation
- OpenAI Integration
- Production Ready Workflow

---

## Workflow

1. Receive customer support request.
2. Send ticket to OpenAI.
3. Classify ticket category.
4. Detect ticket priority.
5. Generate AI summary.
6. Format structured output.
7. Save results into Google Sheets.

---

## Technology Stack

- n8n
- OpenAI API
- Google Sheets
- JSON
- REST API

---

## Workflow Architecture

Customer Support Request

↓

OpenAI Analysis

↓

Ticket Classification

↓

Priority Detection

↓

AI Summary

↓

Google Sheets Logging

---

## Example Input

```json
{
  "customer": "John Smith",
  "email": "john@example.com",
  "subject": "Unable to access my account",
  "message": "I forgot my password and cannot log in."
}
```

---

## Example Output

```json
{
  "category": "Account Issue",
  "priority": "High",
  "summary": "Customer cannot access the account due to password issues."
}
```

---

## Business Value

This workflow enables companies to:

- Reduce manual ticket triage
- Improve response times
- Increase support efficiency
- Standardize ticket classification
- Scale customer support operations

---

## Screenshots

- n8n Workflow
- Google Sheets Output
- Workflow Execution

---

## Future Improvements

- Gmail Integration
- Slack Notifications
- CRM Integration
- Human Approval Workflow
- Automatic Ticket Assignment
- Dashboard & Analytics

---

## Author

Built by **NextWave AI**

AI Automation Portfolio
