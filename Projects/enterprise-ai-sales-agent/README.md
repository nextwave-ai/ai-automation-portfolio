# Enterprise AI Sales Agent

An end-to-end AI-powered B2B sales automation workflow built with n8n and OpenAI.

The system automatically receives inbound leads, validates and analyzes them, scores their sales potential, routes them based on qualification, schedules meetings, generates personalized emails, sends Slack notifications, and logs lead data into Google Sheets.

## Features

- Webhook-based lead intake
- Lead data normalization and validation
- AI-powered lead analysis
- Lead scoring from 0–100
- Cold, Warm, and Hot lead classification
- Buying intent detection
- Lead qualification
- Company and industry analysis
- Estimated deal value classification
- Buying signal detection
- Potential objection analysis
- Automatic lead routing
- Google Calendar meeting scheduling
- AI-generated personalized sales emails
- Automated nurture emails
- Gmail integration
- Slack notifications
- Google Sheets lead logging
- Missing-field error handling
- Automated webhook responses

## Workflow

### Lead Processing

Lead Intake  
→ Normalize Lead Data  
→ Validate Required Fields  
→ AI Lead Analysis  
→ Parse & Validate AI Output  
→ Lead Qualification

### Qualified Leads

Lead Qualification  
→ Meeting Request Check  
→ Google Calendar Event  
→ Generate Personalized Email  
→ Parse Email Content  
→ Send Personalized Email  
→ Slack High Priority Alert  
→ Google Sheets  
→ Success Response

### Nurture Leads

Lead Qualification  
→ Nurture Email  
→ Slack Nurture Queue  
→ Google Sheets  
→ Success Response

### Invalid Leads

Lead Intake  
→ Normalize Lead Data  
→ Validate Required Fields  
→ Missing Required Fields Error

Invalid leads are stopped before AI processing and downstream sales actions.

## AI Lead Intelligence

The AI engine generates structured sales intelligence including:

- Lead Score
- Temperature
- Buying Intent
- Qualification Status
- Industry
- Company Size
- Business Type
- Estimated Deal Value
- Company Summary
- Buying Signals
- Potential Objections
- Recommended Action
- Proposal Angle
- Personalized Opener

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
اصلاح شده را بنویس
# Enterprise AI Sales Agent

An end-to-end AI-powered B2B sales automation workflow built with n8n and OpenAI.

The system automatically receives inbound leads, validates and analyzes them, scores their sales potential, routes them based on qualification, schedules meetings, generates personalized emails, sends Slack notifications, and logs lead data into Google Sheets.

## Features

- Webhook-based lead intake
- Lead data normalization and validation
- AI-powered lead analysis
- Lead scoring from 0–100
- Cold, Warm, and Hot lead classification
- Buying intent detection
- Lead qualification
- Company and industry analysis
- Estimated deal value classification
- Buying signal detection
- Potential objection analysis
- Automatic lead routing
- Google Calendar meeting scheduling
- AI-generated personalized sales emails
- Automated nurture emails
- Gmail integration
- Slack notifications
- Google Sheets lead logging
- Missing-field error handling
- Automated webhook responses

## Workflow

### Lead Processing

Lead Intake  
→ Normalize Lead Data  
→ Validate Required Fields  
→ AI Lead Analysis  
→ Parse & Validate AI Output  
→ Lead Qualification

### Qualified Leads

Lead Qualification  
→ Meeting Request Check  
→ Google Calendar Event  
→ Generate Personalized Email  
→ Parse Email Content  
→ Send Personalized Email  
→ Slack High Priority Alert  
→ Google Sheets  
→ Success Response

### Nurture Leads

Lead Qualification  
→ Nurture Email  
→ Slack Nurture Queue  
→ Google Sheets  
→ Success Response

### Invalid Leads

Lead Intake  
→ Normalize Lead Data  
→ Validate Required Fields  
→ Missing Required Fields Error

Invalid leads are stopped before AI processing and downstream sales actions.

## AI Lead Intelligence

The AI engine generates structured sales intelligence including:

- Lead Score
- Temperature
- Buying Intent
- Qualification Status
- Industry
- Company Size
- Business Type
- Estimated Deal Value
- Company Summary
- Buying Signals
- Potential Objections
- Recommended Action
- Proposal Angle
- Personalized Opener

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
Lead Qualification Logic

The workflow uses AI analysis together with deterministic routing logic.

Score 80–100: Sales Ready
Score 60–79: Qualified
Score 30–59: Needs Nurturing
Score 0–29: Disqualified

Lead temperature:

Hot: 70–100
Warm: 40–69
Cold: 0–39

Leads with a score of 60 or higher are routed to the qualified lead workflow.

Leads below 60 are routed to the nurture workflow.

Tech Stack
n8n — Workflow orchestration
OpenAI API — Lead intelligence and personalized email generation
Gmail — Automated email delivery
Google Calendar — Discovery call scheduling
Google Sheets — Lead pipeline logging
Slack — Sales team notifications
Webhooks — Lead intake and API communication
Main Workflow Nodes
Webhook — Lead Intake
Normalize Lead Data
Validate Required Fields
Error — Missing Required Fields
AI Lead Analysis Engine
Parse & Validate AI Output
Qualify Lead — Score >= 60?
Meeting Requested?
Create an Event
Generate Personalized Email
Parse Email Content
Send Personalized Email
Slack Alert — High Priority Lead
Send Nurture Email
Slack Alert — Nurture Queue
Log to Google Sheets
Respond — Success
Example Lead Input {
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
Tested Scenarios
High-Priority Lead

The workflow was successfully tested with a high-intent lead requesting a sales demo.

Execution path:
Webhook
→ Validation
→ AI Analysis
→ High-Priority Routing
→ Calendar Event
→ Personalized Email
→ Gmail
→ Slack Alert
→ Google Sheets
→ Success Response




Nurture Lead

The workflow was tested with a lead researching AI automation without immediate buying intent.

Execution path:
Webhook
→ Validation
→ AI Analysis
→ Nurture Routing
→ Nurture Email
→ Slack Nurture Queue
→ Google Sheets
→ Success Response
Missing Required Fields

The workflow was tested with incomplete lead data.

Execution path:

Webhook
→ Normalization
→ Validation
→ Missing Required Fields Error

AI processing and downstream sales actions are not executed for invalid leads.
Installation
Download the workflow JSON file from this repository.
Import the workflow into n8n.
Configure your OpenAI credentials.
Configure Gmail credentials.
Configure Google Calendar credentials.
Configure Google Sheets credentials.
Configure Slack credentials.
Connect your lead source to the webhook.
Update node-specific IDs, spreadsheet references, calendars, email addresses, and Slack channels for your environment.
Test all workflow branches before deployment.
Required Google APIs
Depending on your configuration, enable the required APIs in your Google Cloud project:
Google Calendar API
Google Sheets API
Gmail API
Google Drive API
Security
Credentials and secrets are intentionally not included in this repository.
Never commit:
API keys
OAuth client secrets
Access tokens
Refresh tokens
Slack tokens
.env files
Passwords
Private credentials
All integrations must be configured using your own credentials.
Before publishing an exported n8n workflow publicly, review the JSON file for environment-specific identifiers or sensitive information.
Project Goal
This project demonstrates a practical AI automation architecture for B2B sales operations.
Instead of using AI only for text generation, the workflow combines AI analysis, deterministic business logic, workflow orchestration, and external integrations to automate operational sales tasks.
Analyze → Decide → Route → Act → Record
Disclaimer
This project is intended as a portfolio and demonstration implementation.
Before production deployment, additional authentication, monitoring, retry logic, rate limiting, security controls, data privacy measures, and environment-specific configuration should be implemented.

