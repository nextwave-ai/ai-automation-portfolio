# AI Appointment Scheduling Assistant

## Status

⚙️ **Runnable Workflow** — `workflow.json` is included in this folder and can be imported directly into n8n. Tested with 5 sample payloads covering routine, urgent, emergency, out-of-hours, and vague-input scenarios (see `test-payload.json`).

---

## Overview

AI Appointment Scheduling Assistant is an n8n automation workflow that receives incoming appointment requests via webhook, uses OpenAI to analyze and validate the request, classifies its urgency and type, and returns a structured, ready-to-use scheduling recommendation.

It acts as an **intelligent triage and pre-processing layer** for appointment requests — determining urgency, suggesting a time slot, estimating meeting duration, and drafting a client confirmation message — so that a downstream calendar/booking system (or a human) can act on clean, structured data instead of raw, unstructured text.

---

## Business Problem

Manually triaging incoming appointment requests is slow and inconsistent:

- Urgent requests get buried in a queue with routine ones
- Staff manually estimate meeting duration and type
- Vague or incomplete requests (missing time, unclear date) require back-and-forth
- No consistent, professional confirmation message is sent immediately

---

## Solution

This workflow accepts a raw appointment request (name, email, date, time, type, reason, preferred channel), sends it to OpenAI for analysis, validates and sanitizes the AI's output with deterministic code (never trusting the AI response blindly), and routes the response based on urgency — returning a structured JSON payload with a distinct priority flag for urgent/emergency requests.

---

## Workflow Architecture


Webhook (POST /appointment-request)
↓
Normalize Input (Set)
↓
Analyze Request with OpenAI (gpt-4.1-mini)
↓
Validate & Structure Response (Code — safe defaults for every field)
↓
Urgent or Emergency? (IF)
┌─────────────┴─────────────┐
↓                           ↓
Respond — Priority Booking   Respond — Routine Booking
(X-Appointment-Priority: high) (X-Appointment-Priority: normal)


---

## AI Analysis Logic

The OpenAI node classifies each request and returns:

- `urgency`: routine | urgent | emergency
- `validated_date` / `validated_time`: parsed and format-checked
- `is_business_hours`: true/false (Mon–Fri, 09:00–17:00)
- `suggested_slot`: alternative time if the request is outside business hours or unclear
- `appointment_duration_minutes`: estimated based on type and reason
- `classification`: consultation | follow_up | demo | support | onboarding | general
- `channel_recommendation`: video | phone | in_person
- `preparation_notes`: brief notes for the host before the meeting
- `client_confirmation_message`: a ready-to-send confirmation message

**Urgency rules:**
- `emergency` — reason contains words like urgent, critical, emergency, ASAP, immediately
- `urgent` — requested date is today or tomorrow
- `routine` — everything else

### Reliability: AI output is never trusted directly

The `Validate & Structure Response` code node parses the AI's JSON output and applies a strict allow-list and safe fallback for **every single field** (e.g. invalid `urgency` values fall back to `routine`, malformed dates fall back to `null`, missing confirmation messages fall back to a generic one). This prevents a malformed or unexpected AI response from breaking the workflow or reaching the client with bad data.

---

## Technology Stack

- n8n (Webhook, Set, IF, Code, Respond to Webhook nodes)
- OpenAI (gpt-4.1-mini) via LangChain OpenAI node
- JSON / REST (webhook-based API)

---

## Example Input

```json
{
  "client_name": "Sarah Johnson",
  "client_email": "sarah@innovatetech.com",
  "requested_date": "2025-07-15",
  "requested_time": "10:00",
  "appointment_type": "demo",
  "reason": "We want to see a live demo of your enterprise automation platform before making a procurement decision.",
  "preferred_channel": "video"
}


Example Output
{
  "success": true,
  "priority_flag": false,
  "appointment": {
    "urgency": "routine",
    "classification": "demo",
    "validated_date": "2025-07-15",
    "validated_time": "10:00",
    "is_business_hours": true,
    "suggested_slot": "2025-07-15 10:00",
    "appointment_duration_minutes": 60,
    "channel_recommendation": "video",
    "preparation_notes": "Review client's current automation stack before the call.",
    "client_confirmation_message": "Thank you, Sarah — your demo request for July 15th at 10:00 AM is confirmed. Looking forward to showing you the platform!"
  }
}



Five test payloads are included in test-payload.json, covering:

	1.	Standard demo request → routine, ~60 min, video
	2.	Urgent support call (tomorrow) → urgent, priority_flag: true
	3.	Emergency escalation (critical language) → emergency, priority_flag: true
	4.	Out-of-business-hours onboarding request → is_business_hours: false, alternate slot suggested
	5.	Minimal/vague input (no type, vague date) → graceful fallback, no workflow failure

Send any example as a POST request:
curl -X POST https://YOUR-N8N-INSTANCE/webhook/appointment-request \
  -H 'Content-Type: application/json' \
  -d '<payload>'




	•	This workflow does not create a Google Calendar event, send a confirmation email, or check real calendar availability — it only analyzes and structures the request. Calendar/email integration is a natural next step, not yet implemented.
	•	Date/time parsing relies on the AI model rather than a deterministic date-parsing library, so highly ambiguous input (e.g. “sometime next week”) may return null for validated_date.
	•	No persistent storage or logging (e.g. Google Sheets) of incoming requests yet.

Future Improvements

	•	Google Calendar integration for real availability checks and event creation
	•	Automated email/SMS confirmation delivery
	•	Google Sheets logging of all incoming requests
	•	Deterministic date-parsing fallback for ambiguous input
	•	Reschedule/cancel endpoints

Author

Built by NextWave AI
AI Automation Portfolio

