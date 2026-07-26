# AI CRM Data Enrichment & Sales Intelligence Automation

An AI-powered CRM automation workflow built with **n8n**, **OpenAI GPT-4.1**, **Google Sheets**, and **Slack**.

This workflow enriches incoming CRM leads, analyzes their business value using AI, assigns a lead score, stores the results, and instantly alerts the sales team when a high-value opportunity is detected.

---

# Business Problem

Sales teams often receive incomplete CRM records that require manual research before outreach.

This process is slow, inconsistent, and causes valuable leads to be overlooked.

This workflow automates the entire enrichment process in seconds.

---

# Solution

The workflow automatically:

- Receives lead data via Webhook
- Normalizes CRM fields
- Validates required information
- Uses OpenAI GPT-4.1 Mini for sales intelligence
- Calculates Lead Score (0–100)
- Classifies Lead Temperature
- Recommends the next sales action
- Generates AI sales notes
- Stores results in Google Sheets
- Sends Slack alerts for high-value leads
- Returns a structured JSON response

---

# Workflow Architecture

Webhook

↓

Normalize Lead Fields

↓

Validate Email

↓

OpenAI Lead Enrichment

↓

JSON Parsing & Validation

↓

Google Sheets Logging

↓

Lead Score Decision

↓

Slack Notification (High Value Leads)

↓

JSON Response

---

# AI Analysis

The AI generates:

- Company Summary
- Industry Classification
- Company Size
- Business Type
- Lead Score
- Lead Temperature
- Recommended Sales Action
- Sales Notes

---

# Technologies

- n8n
- OpenAI GPT-4.1 Mini
- Google Sheets API
- Slack API
- REST API
- JavaScript
- JSON

---

# Input Example

```json
{
  "firstName":"John",
  "lastName":"Smith",
  "company":"Acme Inc",
  "email":"john@acme.com",
  "website":"https://acme.com",
  "country":"USA",
  "jobTitle":"Sales Director"
}
```

---

# Output Example

```json
{
  "leadScore":92,
  "temperature":"Hot",
  "industry":"SaaS",
  "recommendedAction":"Schedule Demo"
}
```

---

# Key Features

✔ AI Lead Qualification

✔ CRM Data Enrichment

✔ Sales Intelligence

✔ Automatic Lead Scoring

✔ AI Sales Notes

✔ Google Sheets Logging

✔ Slack Alerts

✔ REST API Ready

✔ Error Handling

✔ JSON Validation

---

# Business Benefits

- Reduce manual CRM research
- Prioritize high-value leads
- Improve sales response time
- Standardize lead qualification
- Increase sales productivity

---

# Future Improvements

- HubSpot Integration
- Salesforce Integration
- Pipedrive Integration
- Microsoft Dynamics
- Email Automation
- CRM Update API
- Company Logo Detection
- Clearbit / Apollo Integration

---

# Repository

workflow.json

README.md

screenshots/

LICENSE

---

# Author

AI Automation Portfolio

Built with n8n + OpenAI