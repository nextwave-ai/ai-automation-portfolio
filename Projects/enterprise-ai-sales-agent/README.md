# Enterprise AI Sales Agent

## Status

⚙️ **Runnable Workflow — tested end-to-end.** `Enterprise AI Sales Agent.json` is included in this folder and can be imported directly into n8n. Tested across high-priority, nurture, and missing-field validation paths, including verified Slack notifications.

An end-to-end AI-powered B2B sales automation workflow built with n8n and OpenAI.

The system automatically receives inbound leads, validates and analyzes them, scores their sales potential, routes them based on qualification, schedules meetings, generates personalized emails, sends Slack notifications, and logs lead data into Google Sheets.

## Features

- Webhook-based lead intake
- Lead data normalization and validation (email + company required)
- AI-powered lead analysis (15 structured intelligence fields per lead)
- Lead scoring from 0–100
- Cold, Warm, and Hot lead classification
- Buying intent detection
- Lead qualification (Disqualified / Needs Nurturing / Qualified / Sales Ready)
- Company and industry analysis
- Estimated deal value classification
- Buying signal and objection analysis
- Automatic lead routing by score
- Google Calendar meeting scheduling (for leads that requested a meeting)
- AI-generated personalized sales emails
- Automated nurture emails for lower-scoring leads
- Gmail integration
- Slack notifications (high-priority and nurture queue)
- Google Sheets pipeline logging
- Missing-field error handling
- Automated webhook responses

## Workflow

**Lead Processing:** Webhook — Lead Intake → Normalize Lead Data → Validate Required Fields → AI Lead Analysis Engine → Parse & Validate AI Output → Qualify Lead — Score ≥ 60?

**Qualified Leads (score ≥ 60):** Qualify Lead → Meeting Requested? → [true: Create an Event → Generate Personalized Email] / [false: Generate Personalized Email] → Parse Email Content → Send Personalized Email → Slack Alert — High Priority Lead → Log to Google Sheets → Respond — Success

**Nurture Leads (score < 60):** Qualify Lead → Send Nurture Email → Slack Alert — Nurture Queue → Log to Google Sheets → Respond — Success

**Invalid Leads (missing email or company):** Webhook → Normalize Lead Data → Validate Required Fields → Error — Missing Required Fields (HTTP 400). Invalid leads are stopped before AI processing and downstream sales actions.

## AI Lead Intelligence

The AI engine (OpenAI `gpt-4.1-mini`) generates structured sales intelligence including:

- Lead Score, Temperature, Buying Intent, Qualification Status
- Industry, Company Size, Business Type
- Estimated Deal Value
- Company Summary, Buying Signals, Potential Objections
- Recommended Action, Proposal Angle, Personalized Opener

Example:

```json
{
  "leadScore": 92,
  "temperature": "Hot",
  "intent": "Ready to Buy",
  "qualificationStatus": "Sales Ready",
  "industry": "Unknown",
  "companySize": "11-50",
  "businessType": "B2B",
  "estimatedDealValue": "$25K-$100K",
  "recommendedAction": "Schedule Demo"
}
```

### Reliability: AI output is never trusted directly

The `Parse & Validate AI Output` code node parses the AI's raw JSON response and validates all 15 fields against strict allow-lists. If the AI returns an invalid or missing `temperature` or `qualificationStatus`, both are **derived deterministically from `leadScore`** instead — so the model's classification can never contradict its own numeric score in the final record. Every text field (summary, buying signals, objections, etc.) also falls back to a safe default if missing or too short, so a malformed AI response can never break the workflow or reach a client with empty data.

## Lead Qualification Logic

The workflow uses AI analysis together with deterministic routing logic.

- Score 80–100: Sales Ready
- Score 60–79: Qualified
- Score 30–59: Needs Nurturing
- Score 0–29: Disqualified

Lead temperature: Hot 70–100 | Warm 40–69 | Cold 0–39

Leads with a score of 60 or higher are routed to the qualified lead workflow. Leads below 60 are routed to the nurture workflow.

## Tech Stack

- n8n — Workflow orchestration
- OpenAI API (gpt-4.1-mini) — Lead intelligence and personalized email generation
- Gmail — Automated email delivery
- Google Calendar — Discovery call scheduling
- Google Sheets — Lead pipeline logging
- Slack — Sales team notifications
- Webhooks — Lead intake and API communication

## Main Workflow Nodes

Webhook — Lead Intake → Normalize Lead Data → Validate Required Fields → Error — Missing Required Fields → AI Lead Analysis Engine → Parse & Validate AI Output → Qualify Lead — Score ≥ 60? → Meeting Requested? → Create an Event → Generate Personalized Email → Parse Email Content → Send Personalized Email → Slack Alert — High Priority Lead → Send Nurture Email → Slack Alert — Nurture Queue → Log to Google Sheets → Respond — Success

## Example Lead Input

```json
{
  "firstName": "Sarah",
  "lastName": "Mitchell",
  "fullName": "Sarah Mitchell",
  "email": "sarah@example.com",
  "company": "TechFlow Solutions",
  "jobTitle": "Head of Sales",
  "website": "https://example.com",
  "country": "Austria",
  "message": "We are evaluating AI sales automation for our B2B sales team. We need automated lead qualification, personalized outreach, CRM integration, and follow-up automation. We would like to schedule a demo and discuss implementation.",
  "meetingRequested": true,
  "source": "website"
}
```

## Tested Scenarios

**High-Priority Lead** — tested with a high-intent lead requesting a sales demo. Path: Webhook → Validation → AI Analysis → High-Priority Routing → Calendar Event → Personalized Email → Gmail → Slack Alert → Google Sheets → Success Response.

**Nurture Lead** — tested with a lead researching AI automation without immediate buying intent. Path: Webhook → Validation → AI Analysis → Nurture Routing → Nurture Email → Slack Nurture Queue → Google Sheets → Success Response. Slack alert correctly displays the lead's real name and company.

**Missing Required Fields** — tested with incomplete lead data. Path: Webhook → Normalization → Validation → Missing Required Fields Error. AI processing and downstream sales actions are not executed for invalid leads.

## Installation

1. Download the workflow JSON file from this repository.
2. Import the workflow into n8n.
3. Configure your OpenAI, Gmail, Google Calendar, Google Sheets, and Slack credentials.
4. Connect your lead source to the webhook.
5. Update node-specific IDs, spreadsheet references, calendars, email addresses, and Slack channels for your environment.
6. Test all workflow branches before deployment.

## Required Google APIs

Depending on your configuration, enable the required APIs in your Google Cloud project: Google Calendar API, Google Sheets API, Gmail API, Google Drive API.

## Security

Credentials and secrets are intentionally not included in this repository. Never commit API keys, OAuth client secrets, access tokens, refresh tokens, Slack tokens, `.env` files, passwords, or private credentials. All integrations must be configured using your own credentials. Before publishing an exported n8n workflow publicly, review the JSON file for environment-specific identifiers or sensitive information.

## Limitations

- AI-derived fields like industry and company size are inferred from limited input, not looked up from a verified data source.
- No retry logic if the Gmail, Calendar, Slack, or Sheets API call fails mid-workflow.
- No deduplication: the same lead submitted twice will be processed and logged twice.

## Project Goal

This project demonstrates a practical AI automation architecture for B2B sales operations. Instead of using AI only for text generation, the workflow combines AI analysis, deterministic business logic, workflow orchestration, and external integrations to automate operational sales tasks.

Analyze → Decide → Route → Act → Record

## Disclaimer

This project is intended as a portfolio and demonstration implementation. Before production deployment, additional authentication, monitoring, retry logic, rate limiting, security controls, data privacy measures, and environment-specific configuration should be implemented.