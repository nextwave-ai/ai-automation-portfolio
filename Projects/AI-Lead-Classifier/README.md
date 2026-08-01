# AI Lead Classifier

> **Automatically classify, score, and prioritize incoming leads using AI-powered workflow automation built with n8n and OpenAI.**

<p align="center">
  <img src="assets/hero.png" alt="AI Lead Classifier">
</p>

<p align="center">

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-ff6d00)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-10a37f)
![License](https://img.shields.io/badge/License-MIT-blue)
![Status](https://img.shields.io/badge/Status-Production_Ready-success)

</p>

---

# Overview

AI Lead Classifier is a production-ready n8n workflow that automatically analyzes incoming leads using OpenAI, classifies their intent, assigns a priority score, and stores structured results for sales teams.

Instead of manually reviewing every inquiry, the workflow instantly determines which leads deserve immediate attention, allowing sales representatives to focus on high-value opportunities.

The automation is designed for agencies, SaaS businesses, consultants, and companies receiving large volumes of inbound leads.

---

# Business Problem

Sales teams often waste valuable time manually reviewing contact forms, emails, and inquiries before determining which prospects are worth pursuing.

Common challenges include:

- Manual lead qualification
- Slow response times
- Inconsistent prioritization
- Human bias during evaluation
- Lost revenue from delayed follow-up
- Poor CRM data quality

As the number of incoming leads grows, manual qualification quickly becomes inefficient and difficult to scale.

---

# Solution

AI Lead Classifier automates the lead qualification process using artificial intelligence.

When a new lead is received, the workflow extracts customer information, sends the data to OpenAI for analysis, determines the lead category and priority, and stores the structured output for future sales activities.

This enables businesses to respond faster, improve qualification accuracy, and reduce repetitive manual work.

---

# Key Features

- AI Lead Qualification
- Lead Priority Scoring
- Customer Intent Classification
- Structured JSON Output
- Google Sheets Integration
- Automated Workflow Execution
- Modular n8n Architecture
- Error Handling
- Production Ready Design

---

# Workflow Architecture

```text
New Lead

      │

      ▼

Webhook / Form

      │

      ▼

Data Validation

      │

      ▼

OpenAI Analysis

      │

      ▼

Lead Classification

      │

      ▼

Priority Assignment

      │

      ▼

Google Sheets

      │

      ▼

Workflow Complete
```

---

# Technology Stack

| Technology | Purpose |
|------------|---------|
| n8n | Workflow Automation |
| OpenAI API | AI Lead Classification |
| Google Sheets API | Lead Storage |
| Webhooks | Data Collection |
| JSON | Structured Data |
| REST API | External Integrations |

---

# Workflow Process

1. Receive a new lead.
2. Validate customer information.
3. Extract relevant lead data.
4. Analyze the content using OpenAI.
5. Determine customer intent.
6. Assign a lead category.
7. Calculate priority level.
8. Store structured information.
9. Complete the workflow.

---

# Example Input

```json
{
  "name": "John Smith",
  "email": "john@example.com",
  "company": "Acme Inc.",
  "message": "We need CRM automation for our sales team."
}
```

---

# Example Output

```json
{
  "category": "Sales",
  "priority": "High",
  "intent": "CRM Automation",
  "recommendedAction": "Contact within 1 hour"
}
```

---

# Business Impact

Organizations using this workflow can:

- Reduce manual lead qualification time
- Respond to high-value leads faster
- Standardize lead evaluation
- Improve CRM data quality
- Increase sales productivity
- Scale lead management without additional administrative work

---

# Screenshots

```text
assets/

├── hero.png
├── workflow.png
├── execution.png
├── google-sheets.png
└── architecture.png
```

---

# Project Structure

```text
AI-Lead-Classifier/

├── README.md
├── workflow.json
├── .env.example
├── sample-input.json
├── sample-output.json
├── assets/
│   ├── hero.png
│   ├── workflow.png
│   ├── execution.png
│   ├── architecture.png
│   └── google-sheets.png
└── LICENSE
```

---

# Installation & Setup

1. Clone the repository.
2. Import `workflow.json` into n8n.
3. Configure the required credentials.
4. Add the environment variables.
5. Execute the workflow.

---

# Environment Variables

```env
OPENAI_API_KEY=

GOOGLE_CLIENT_ID=

GOOGLE_CLIENT_SECRET=

GOOGLE_REFRESH_TOKEN=
```

---

# Future Improvements

- CRM integrations (HubSpot, Salesforce)
- Airtable support
- Automatic email notifications
- Lead enrichment APIs
- Slack and Microsoft Teams integration
- AI lead scoring model customization
- Analytics dashboard
- Multi-language lead classification

---

# License

This project is licensed under the MIT License.

---

# Author

**NextWave AI**

Production-ready AI Automation workflows built with **n8n**, **OpenAI**, and modern workflow automation technologies.