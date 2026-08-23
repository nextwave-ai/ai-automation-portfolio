# AI-GHL-Lead-Qualifier

AI-powered lead qualification and CRM automation built on **n8n** and **GoHighLevel (GHL)**. The workflow listens for new form submissions in GHL, uses OpenAI to evaluate and qualify each inquiry, then automatically tags the contact in the CRM and sends a personalized follow-up email — all without manual review.

This project bridges GoHighLevel's native CRM/marketing layer with custom AI logic via n8n, rather than relying on GHL's built-in (and more limited) automation tools alone.

---

## The Problem

Businesses using GoHighLevel to capture leads from website forms typically respond to every inquiry the same way — or worse, respond manually, which doesn't scale. Every submission gets treated identically, whether it's a genuine, high-value business inquiry or spam/noise. This wastes time on low-value leads and delays response time on the leads that actually matter.

## The Solution

This workflow automatically:
1. Captures a new inquiry the moment a GHL form is submitted
2. Uses an LLM (GPT-4o-mini) to read the inquiry and decide whether it's a genuine, qualified lead
3. Tags the contact in GHL accordingly (`ai-qualified-high-value` or `ai-low-value-or-spam`)
4. For qualified leads, immediately sends a personalized follow-up email drafted by the AI — before a human ever looks at it

## Architecture

```
GHL Form Submission
        |
        v
GHL Workflow (native Trigger + Webhook Action)
        |
        v
n8n Webhook (receives contact + inquiry data)
        |
        v
OpenAI (gpt-4o-mini) — reads the inquiry, returns structured JSON:
   { qualified, category, reason, suggested_reply }
        |
        v
Parse JSON response into usable fields
        |
        v
   IF qualified == true
        |
   ------------------------
   |                      |
   v                      v
TRUE branch            FALSE branch
Tag contact:            Tag contact:
"ai-qualified-          "ai-low-value-
 high-value"             or-spam"
   |
   v
Send follow-up email via GHL
(using the AI-generated suggested_reply)
```

## Tech Stack

- **n8n** — self-hosted workflow orchestration engine
- **GoHighLevel (GHL)** — CRM, forms, and contact/conversation management
- **OpenAI API (gpt-4o-mini)** — lead qualification and reply generation
- **GHL REST API v2** — programmatic contact tagging and email sending, authenticated via a GHL Private Integration token

## How It Works — Node by Node

| Node | Purpose |
|---|---|
| **Webhook** | Receives the raw form submission payload from a native GHL Workflow (Form Submitted trigger → Webhook action) |
| **Message a model (OpenAI)** | System prompt instructs the model to act as a lead qualifier and return strict JSON only: `qualified` (boolean), `category`, `reason`, `suggested_reply` |
| **Edit Fields** | Parses the OpenAI response (returned as a JSON string) into a real, usable object |
| **If** | Branches on `parsed.qualified` |
| **HTTP Request (True branch)** | POSTs to the GHL Contacts API to add the `ai-qualified-high-value` tag |
| **HTTP Request1 (True branch)** | POSTs to the GHL Conversations API to send the AI's drafted follow-up email |
| **HTTP Request2 (False branch)** | POSTs to the GHL Contacts API to add the `ai-low-value-or-spam` tag — no email sent |

## Example Output

**Qualified inquiry:**
> "I manage a dental clinic and want to set up automated appointment reminders and lead follow-up for new patient inquiries from our website."

Result: `qualified: true`, tagged `ai-qualified-high-value`, follow-up email sent automatically.

**Low-value / spam submission:**
> "asdfasdf blah blah nothing"

Result: `qualified: false`, tagged `ai-low-value-or-spam`, no email sent.

## Screenshots

**Qualified and unqualified leads tagged automatically in GHL:**
![GHL Contacts tagged by AI](screenshots/01-ghl-contacts-tagged.png)

**Automated follow-up email delivered to a qualified lead:**
![Email sent to qualified lead](screenshots/02-ghl-email-sent.png)

**n8n workflow — qualified lead (true branch executed):**
![n8n true branch](screenshots/03-n8n-workflow-true-branch.png)

**n8n workflow — spam/unqualified lead (false branch executed):**
![n8n false branch](screenshots/04-n8n-workflow-false-branch.png)

## Notes on This Project

This was built as a hands-on capability validation project against a live GoHighLevel account (not a simulated demo), to confirm this exact integration pattern — GHL + n8n + AI — works end-to-end before offering it to clients. Both branches (qualified and unqualified) were tested with real form submissions through a live GHL form, not mock data.

The local n8n instance in this build was tunneled via ngrok for testing purposes. A production deployment for a real client would run on a persistent, always-on n8n instance (self-hosted or n8n Cloud) rather than a local machine.

## Related Projects

This project extends the CRM/lead-automation patterns used in:
- `AI-CRM-Automation`
- `AI-Lead-Classifier`
- `AI-CRM-Data-Enrichment`

---

Built by **NextWave AI** — AI Automation & Integration Specialist (n8n + AI + CRM/GoHighLevel + APIs).