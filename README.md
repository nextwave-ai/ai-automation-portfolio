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
- Capability Validation (🔬) — built and tested end-to-end against a live third-party platform account (not a simulated demo, not mock data), to confirm a specific integration pattern works before offering it to clients. Hosted as a separate repository. See that project's own README for the platform-specific context.

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

## 9. AI RAG Knowledge Base & Business Agent

Projects/AI-RAG-Knowledge-Base-Agent/ — ⚙️ Runnable Workflow

A retrieval-augmented generation (RAG) agent that answers business questions grounded in company documents, and takes real action when the user's intent requires it. Two independent workflows: a document ingestion pipeline that chunks and embeds text into a Google Sheets-based vector store, and a query agent that retrieves relevant context via cosine similarity, answers questions with source attribution, books meetings in Google Calendar when asked, and escalates low-confidence questions to Slack instead of letting the model guess.

Ingestion: Webhook → Chunk Text → Generate Embedding → Append to Google Sheet
Query: Webhook → Embed Question → Semantic Search (cosine similarity) → Confidence Check → [Generate Answer → Intent Router → Book Meeting or Respond] / [Notify Human via Slack]

## 10. AI WhatsApp Sales & Support Agent

Projects/AI-WhatsApp-Business-Agent/ — ⚙️ Runnable Workflow

An AI agent that receives real WhatsApp messages via the Meta Cloud API, classifies each one as a sales or support inquiry, and replies directly on WhatsApp. Support questions are routed to the AI RAG Knowledge Base project's own webhook rather than duplicating a second knowledge base, so one set of company documents serves both a direct API channel and WhatsApp. Every inbound and outbound message is logged to Google Sheets. The sales path is confirmed working end-to-end with a real message delivered back to a live phone; the support path is implemented and integrated but requires the RAG project's query workflow to be running alongside this one.

Meta WhatsApp Cloud API → Webhook (verify + receive) → Extract Message → Log Inbound → Classify Intent (AI) → [support: Query Knowledge Base (RAG project webhook)] / [sales: Generate Sales Reply (AI)] → Send WhatsApp Reply (Graph API) → Log Outbound

---

# Capability Validation Projects

These are built and tested differently from the Featured Projects above: end-to-end against a live account on a real third-party platform (not a simulated demo, not mock data), specifically to confirm a particular integration pattern actually works before offering it to clients. Each is hosted as its own separate repository rather than a subfolder of this one.

## 11. AI GHL Lead Qualifier

[github.com/nextwave-ai/AI-GHL-Lead-Qualifier](https://github.com/nextwave-ai/AI-GHL-Lead-Qualifier) — 🔬 Capability Validation

Bridges GoHighLevel (GHL) — a CRM/marketing platform with limited native automation — with custom AI logic via n8n. When a new inquiry comes in through a GHL form, an AI model reads and qualifies it, then the contact is automatically tagged in GHL and, for genuinely qualified leads, sent a personalized AI-drafted follow-up email — without manual review. Built and tested against a live GHL trial account, with both the qualified and unqualified paths confirmed using real form submissions.

GHL Form Submission → GHL Workflow (Webhook Action) → n8n Webhook → AI Qualification (OpenAI) → Parse → [Qualified: Tag Contact + Send Follow-up Email] / [Unqualified: Tag Contact Only]

---

# Technology Stack

Workflow Automation: n8n

Artificial Intelligence: OpenAI API (gpt-4.1-mini, gpt-4o-mini, text-embedding-3-small), LangChain nodes (LLM Chain, Chat Model, Output Parser), structured/validated AI outputs, prompt engineering, retrieval-augmented generation (RAG)

Google Workspace: Gmail, Google Calendar, Google Sheets

CRM Platforms: GoHighLevel (GHL) — REST API v2, Private Integrations, Contacts, Conversations

Communication: Slack, Telegram, WhatsApp Business Cloud API (Meta)

APIs & Data: REST APIs, Webhooks, JSON, HTTP Request

Development: JavaScript (Code nodes), conditional routing, deterministic validation logic

---

# Skills Demonstrated

- AI workflow automation and prompt engineering
- Business process, sales, and CRM automation
- Lead qualification and enrichment
- AI decision systems with deterministic validation layers
- Webhook and Google Sheets-triggered architectures
- Retrieval-augmented generation (RAG) and semantic search
- Multichannel conversational AI (WhatsApp Business Cloud API integration)
- API integration (OpenAI, Google Workspace, Slack, Telegram, Meta WhatsApp, GoHighLevel)
- Conditional workflow routing and error handling
- Data validation and normalization
- Structured AI output design (allow-lists, safe defaults, placeholder rejection)
- Multi-system integration and workflow testing
- Technical documentation and honest project status reporting

---

# Business Use Cases

These architectures can be adapted for B2B sales teams, marketing agencies, SaaS companies, consulting and service businesses, CRM management (including GoHighLevel), lead generation, customer support, sales operations, client onboarding, internal knowledge management, WhatsApp-based customer engagement, and internal operations automation.

---

# Repository Structure

Projects/ contains: AI-Appointment-Scheduling-Assistant, AI-Appointment-Setter-Sheet, AI-CRM-Automation, AI-CRM-Data-Enrichment, AI-Customer-Support-Ticket-Classifier, AI-Lead-Classifier, AI-Proposal-Generator, AI-RAG-Knowledge-Base-Agent, AI-WhatsApp-Business-Agent, Enterprise-AI-Sales-Agent — plus README.md and LICENSE at the repository root.

Each project folder contains a README.md (business problem, architecture, setup, limitations), a workflow.json where available, and screenshots as supporting evidence.

Capability Validation projects (see above) are hosted as separate, standalone repositories rather than subfolders here, since they're built against live third-party accounts rather than portfolio demos — currently: AI-GHL-Lead-Qualifier.

---

# Workflow Design Principles

1. Validate Before AI Processing — invalid or incomplete data is rejected before any AI or external API call is made.

2. Structured AI Output, Never Trusted Directly — AI responses use structured JSON, and downstream code nodes validate every field against an allow-list with safe defaults, so a malformed or inconsistent AI response can never break the workflow or reach a client with bad data.

3. AI + Deterministic Logic — AI handles classification, analysis, and personalization; deterministic logic handles routing, validation, integrations, and business actions.

4. Business-Focused Automation — each workflow solves a recognizable operational problem, not just a technical demonstration.

5. Honest Status Reporting — a project is only called "runnable" if an importable, tested workflow.json is included. Screenshots alone are labeled as a documented concept, not a working deliverable. Where only part of a workflow's paths are fully confirmed end-to-end, that distinction is stated explicitly in the project's own README rather than implied. Capability Validation projects are labeled as such rather than folded into the main portfolio, since they're built against a personal trial account rather than a delivered client project.

6. Modular Architecture: Input → Validation → Processing → AI Analysis → Decision → Action → Logging → Response

---

# Installation

1. Download the workflow.json for the project you want (where available — see status badges above).
2. Import it into n8n.
3. Configure your own credentials (OpenAI, Google Workspace, Slack, Telegram, Meta WhatsApp, GoHighLevel as needed).
4. Update environment-specific IDs — spreadsheet IDs, calendar IDs, Slack channels, phone number IDs, webhook paths.
5. Test every workflow branch before relying on it.

---

# Security

Credentials and secrets are intentionally not included in this repository. Never commit API keys, OAuth client secrets, access tokens, refresh tokens, Slack/Telegram/Meta/GoHighLevel tokens, .env files, or passwords. All integrations must be configured using your own credentials and environment. Exported workflow files should always be reviewed for sensitive or environment-specific information before being published publicly.

---

# Project Status

The projects in this repository are portfolio and demonstration implementations. Most have been tested end-to-end across multiple execution paths, including success, error, and edge-case scenarios — see each project's own README for its specific test coverage.

Portfolio-tested does not mean production-ready. Production deployment would require additional work: authentication, rate limiting, retry strategies, centralized error handling, monitoring, duplicate protection beyond simple key-matching, API failure handling, data privacy controls, environment management, and security hardening.

---

# Future Development

Future projects will prioritize technical capabilities not yet represented in this portfolio, rather than repeating existing patterns:

- External REST API integrations with authentication, pagination, and retry logic
- AI agents with tool calling and multi-step reasoning
- Voice AI conversational agents
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

n8n · OpenAI · Google Workspace · Slack · Telegram · Meta WhatsApp Business Cloud API · GoHighLevel · REST APIs · Webhooks · JavaScript · JSON

---

If you find these projects useful, consider giving the repository a star.