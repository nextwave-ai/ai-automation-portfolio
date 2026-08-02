# AI Automation Portfolio

A portfolio of AI-powered business automation systems built with n8n, OpenAI, Google Workspace, Slack, and Telegram.

Each project combines AI reasoning with deterministic workflow logic — validation, conditional routing, structured data processing, and automated business actions — to build systems that can:

Receive → Analyze → Decide → Route → Act → Record

This is not a collection of isolated AI prompts. Every workflow is designed around a real operational problem, with explicit attention to error handling, data validation, and — where the AI model's output feeds a downstream action — defensive parsing that never lets a malformed AI response break the workflow or reach a client with bad data.

---

## Project Status Legend

Not every project in this portfolio is at the same maturity level, and that's stated explicitly rather than implied:

- Runnable Workflow (⚙️) — a working workflow.json is included in the project folder, importable directly into n8n, and has been tested end-to-end.
- Documented Concept (📄) — the workflow was built and executed in n8n (screenshots included as evidence), but an importable workflow.json is not yet included in this repository.

No project in this repository is labeled "production-ready." See Project Status below for what that distinction actually means.

---

# Featured Projects

## 1. Enterprise AI Sales Agent

Projects/Enterprise-AI-Sales-Agent/ — ⚙️ Runnable Workflow

An end-to-end B2B sales automation system. Inbound leads are validated, analyzed by AI across 15 structured intelligence fields (score, temperature, buying intent, qualification, industry, deal value, objections, and more), and routed to either a high-priority path (Calendar booking + personalized AI-generated email + Slack alert) or a nurture path (automated nurture email + Slack queue notification). Every lead is logged to Google Sheets regardless of path.

Webhook Lead Intake → Normalize → Validate Required Fields → AI Lead Analysis → Parse & Validate AI Output → Qualify Lead (score ≥ 60?) → [Qualified: Meeting Check → Calendar → Personalized Email → Gmail → Slack] / [Nurture: Nurture Email → Slack] → Google Sheets → Response

## 2. AI CRM Data Enrichment

Projects/AI-CRM-Data-Enrichment/ — ⚙️ Runnable Workflow

A webhook-based CRM enrichment workflow. Every incoming lead is validated, enriched by AI across 8 dimensions (industry, company size, business type, deal value, buying signals, objections), logged to Google Sheets unconditionally, and — for leads scoring 80 or above — instantly flagged to the sales team via Slack.

Webhook → Normalize → Email Present? → AI Enrichment → Parse & Validate → Google Sheets → High Value Lead? → [Slack Alert] → Response

## 3. AI CRM Automation

Projects/AI-CRM-Automation/ — ⚙️ Runnable Workflow

A passive, sheet-driven companion to the webhook-based CRM Enrichment project above. Instead of receiving leads via API, this workflow watches a Google Sheet directly — any team member can drop in a new row with no integration work, and the lead is automatically scored and enriched in place. Updates are matched by email, so re-running the workflow never creates duplicate rows.

Google Sheets Trigger (new row) → Normalize → AI Enrichment (LangChain LLM Chain) → Validate & Reject Placeholders → Update Row (matched by Email)

## 4. AI Appointment Scheduling Assistant

Projects/AI-Appointment-Scheduling-Assistant/ — ⚙️ Runnable Workflow

An intelligent triage and pre-processing layer for appointment requests. Rather than booking meetings directly, it analyzes each request with AI to determine urgency (routine / urgent / emergency), suggest a time slot, estimate meeting duration, and draft a client confirmation message — producing clean, structured data for a downstream booking system or human to act on.

Webhook → Normalize → AI Analysis → Validate & Structure Response → Urgent or Emergency? → [Priority Response] / [Routine Response]

## 5. AI Appointment Setter Sheet

Projects/AI-Appointment-Setter-Sheet/ — ⚙️ Runnable Workflow

Drafts a personalized outreach email for every new lead added to a Google Sheet. The design deliberately separates the AI-written part (the email body) from a fixed, human-edited sender signature — so the signature can never contain an unfilled template placeholder, a failure mode this project was specifically rebuilt to eliminate.

Google Sheets Trigger → Normalize → Sender Info (fixed values) → AI Email Body → Reject Placeholders & Append Signature → Update Row

## 6. AI Customer Support Ticket Classifier

Projects/AI-Customer-Support-Ticket-Classifier/ — ⚙️ Runnable Workflow

Automatically analyzes incoming support requests, classifies intent, assigns priority, and returns a structured result for downstream routing — reducing manual ticket triage.

## 7. AI Lead Classifier

Projects/AI-Lead-Classifier/ — 📄 Documented Concept

Scores company leads on a 0–10 scale with a written justification, using a Google Sheets-triggered LLM chain, and sends an instant Telegram alert for scores above 7. Built and executed in n8n (see included Google Sheets and Telegram screenshots); an importable workflow.json is not yet included — see the project's Setup section to rebuild it.

## 8. AI Proposal Generator

Projects/AI-Proposal-Generator/ — 📄 Documented Concept

Generates a structured client proposal (scope, pricing, timeline) from project requirements using OpenAI. Built and executed in n8n (see included screenshots); an importable workflow.json is not yet included in this repository.

---

# Technology Stack

Workflow Automation: n8n

Artificial Intelligence: OpenAI API (gpt-4.1-mini), LangChain nodes (LLM Chain, Chat Model, Output Parser), structured/validated AI outputs, prompt engineering

Google Workspace: Gmail, Google Calendar, Google Sheets

Communication: Slack, Telegram

APIs & Data: REST APIs, Webhooks, JSON, HTTP Request

Development: JavaScript (Code nodes), conditional routing, deterministic validation logic

---

# Skills Demonstrated

- AI workflow automation and prompt engineering
- Business process, sales, and CRM automation
- Lead qualification and enrichment
- AI decision systems with deterministic validation layers
- Webhook and Google Sheets-triggered architectures
- API integration (OpenAI, Google Workspace, Slack, Telegram)
- Conditional workflow routing and error handling
- Data validation and normalization
- Structured AI output design (allow-lists, safe defaults, placeholder rejection)
- Multi-system integration and workflow testing
- Technical documentation and honest project status reporting

---

# Business Use Cases

These architectures can be adapted for B2B sales teams, marketing agencies, SaaS companies, consulting and service businesses, CRM management, lead generation, customer support, sales operations, client onboarding, and internal operations automation.

---

# Repository Structure

Projects/ contains: AI-Appointment-Scheduling-Assistant, AI-Appointment-Setter-Sheet, AI-CRM-Automation, AI-CRM-Data-Enrichment, AI-Customer-Support-Ticket-Classifier, AI-Lead-Classifier, AI-Proposal-Generator, Enterprise-AI-Sales-Agent — plus README.md and LICENSE at the repository root.

Each project folder contains a README.md (business problem, architecture, setup, limitations), a workflow.json where available, and screenshots as supporting evidence.

---

# Workflow Design Principles

1. Validate Before AI Processing — invalid or incomplete data is rejected before any AI or external API call is made.

2. Structured AI Output, Never Trusted Directly — AI responses use structured JSON, and downstream code nodes validate every field against an allow-list with safe defaults, so a malformed or inconsistent AI response can never break the workflow or reach a client with bad data.

3. AI + Deterministic Logic — AI handles classification, analysis, and personalization; deterministic logic handles routing, validation, integrations, and business actions.

4. Business-Focused Automation — each workflow solves a recognizable operational problem, not just a technical demonstration.

5. Honest Status Reporting — a project is only called "runnable" if an importable, tested workflow.json is included. Screenshots alone are labeled as a documented concept, not a working deliverable.

6. Modular Architecture: Input → Validation → Processing → AI Analysis → Decision → Action → Logging → Response

---

# Installation

1. Download the workflow.json for the project you want (where available — see status badges above).
2. Import it into n8n.
3. Configure your own credentials (OpenAI, Google Workspace, Slack, Telegram as needed).
4. Update environment-specific IDs — spreadsheet IDs, calendar IDs, Slack channels, webhook paths.
5. Test every workflow branch before relying on it.

---

# Security

Credentials and secrets are intentionally not included in this repository. Never commit API keys, OAuth client secrets, access tokens, refresh tokens, Slack/Telegram tokens, .env files, or passwords. All integrations must be configured using your own credentials and environment. Exported workflow files should always be reviewed for sensitive or environment-specific information before being published publicly.

---

# Project Status

The projects in this repository are portfolio and demonstration implementations. Most have been tested end-to-end across multiple execution paths, including success, error, and edge-case scenarios — see each project's own README for its specific test coverage.

Portfolio-tested does not mean production-ready. Production deployment would require additional work: authentication, rate limiting, retry strategies, centralized error handling, monitoring, duplicate protection beyond simple key-matching, API failure handling, data privacy controls, environment management, and security hardening.

---

# Future Development

Future projects will prioritize technical capabilities not yet represented in this portfolio, rather than repeating existing patterns:

- External REST API integrations with authentication, pagination, and retry logic
- Document processing and RAG / vector search systems
- AI agents with tool calling and multi-step reasoning
- Human-in-the-loop approval workflows
- Sub-workflows and reusable components

---

# Portfolio Goal

To demonstrate the ability to design and build practical AI automation systems that connect AI models with real business tools — moving beyond isolated AI prompts toward complete systems that Analyze → Decide → Route → Act → Record.

---

# Contact

Open to freelance and project opportunities involving AI automation, n8n development, OpenAI integrations, CRM and sales automation, and business workflow automation. Contact information can be found through my GitHub profile.

---

# Built With

n8n · OpenAI · Google Workspace · Slack · Telegram · REST APIs · Webhooks · JavaScript · JSON

---

If you find these projects useful, consider giving the repository a star.