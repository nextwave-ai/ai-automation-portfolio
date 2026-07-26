# AI Appointment Scheduling Assistant
### n8n + OpenAI GPT-4.1-mini — Production Ready — v1.0

Automatically receives appointment requests, validates date and time, classifies urgency, recommends the best meeting slot and channel, and routes the response — all within a single webhook call.

---

## Overview

Booking appointments manually is slow, inconsistent, and hard to scale. This automation handles the entire intake and analysis layer: a client submits a request, the AI parses it, validates the timing, estimates how long the meeting should be, decides whether it is routine or urgent, and returns a structured response with everything the host needs to confirm the booking.

The workflow splits into two branches at the routing layer — urgent and emergency requests get a priority-flagged response with a dedicated HTTP header, while routine bookings follow the standard path. Both paths return identical data shapes, making downstream integration predictable.

---

## Features

- **Date & Time Validation** — parses natural-language date inputs into standardized `YYYY-MM-DD` and `HH:MM` formats; returns `null` if unparseable so downstream systems can handle gracefully
- **Business Hours Check** — detects whether the requested slot falls within Monday–Friday 09:00–17:00 and flags out-of-hours requests
- **Smart Slot Suggestion** — if the request is outside business hours or the time is unclear, the AI suggests the next available appropriate slot
- **Urgency Classification** — classifies every request as `routine`, `urgent`, or `emergency` based on the reason text and proximity of the requested date
- **Meeting Type Classification** — categorizes the appointment as `consultation`, `follow_up`, `demo`, `support`, `onboarding`, or `general`
- **Duration Estimation** — infers the appropriate meeting length in minutes based on appointment type and stated reason
- **Channel Recommendation** — recommends `video`, `phone`, or `in_person` based on the request context
- **Preparation Notes** — generates a 1–2 sentence brief for the host to read before the meeting
- **Client Confirmation Message** — produces a ready-to-send confirmation message for the client
- **Priority Routing** — urgent and emergency bookings are separated at the IF node and flagged with `priority_flag: true` and an `X-Appointment-Priority: high` response header
- **Fault-Tolerant Parsing** — every output field is validated against allowed values with safe defaults applied if the AI response is incomplete

---

## Workflow Architecture

```
POST /webhook/appointment-request
          │
          ▼
    [Webhook]
    Receives raw appointment payload
          │
          ▼
    [Normalize Input]  ← Set node (typeVersion 3.4)
    Maps 8 fields: client_name, client_email,
    requested_date, requested_time, appointment_type,
    reason, preferred_channel, received_at (auto-timestamped)
          │
          ▼
    [Analyze Request with OpenAI]  ← GPT-4.1-mini
    Validates date/time, estimates duration,
    detects urgency, suggests slot, writes confirmation
          │
          ▼
    [Validate & Structure Response]  ← Code node
    Safe JSON parse with try/catch
    Validates all 10 output fields
    Applies safe defaults for missing values
          │
          ▼
    [Urgent or Emergency?]  ← IF node
    Condition: urgency != "routine"
          │
    ┌─────┴──────┐
    │ TRUE       │ FALSE
    ▼            ▼
[Priority    [Routine
 Booking]     Booking]
 flag: true   flag: false
 header: high header: normal
```

---

## Node Reference

| # | Node Name | Type | Version | Purpose |
|---|---|---|---|---|
| 1 | Webhook | `n8n-nodes-base.webhook` | 2 | HTTP entry point |
| 2 | Normalize Input | `n8n-nodes-base.set` | 3.4 | Input normalization |
| 3 | Analyze Request with OpenAI | `@n8n/n8n-nodes-langchain.openAi` | 1.8 | AI analysis |
| 4 | Validate & Structure Response | `n8n-nodes-base.code` | 2 | Parse & validate |
| 5 | Urgent or Emergency? | `n8n-nodes-base.if` | 2.2 | Priority routing |
| 6 | Respond — Priority Booking | `n8n-nodes-base.respondToWebhook` | 1.1 | Urgent response |
| 7 | Respond — Routine Booking | `n8n-nodes-base.respondToWebhook` | 1.1 | Standard response |

---

## Import Instructions

### Prerequisites
- n8n version **2.27 or higher**
- OpenAI API account with **gpt-4.1-mini** access

### Step 1 — Import

1. Open your n8n instance
2. **Workflows → Add Workflow → Import from File**
3. Upload `workflow.json`
4. Click **Save**

### Step 2 — Add OpenAI Credential

1. Click the **"Analyze Request with OpenAI"** node
2. **Credential → Create New → OpenAI**
3. Paste your API key (`sk-...`)
4. Click **Save**

### Step 3 — Activate

1. Toggle the workflow to **Active** (top-right)
2. Copy the **Production Webhook URL** from the Webhook node:
   ```
   https://your-n8n-instance.com/webhook/appointment-request
   ```

### Step 4 — Test

Send a POST request using the payloads in `test-payload.json`.

---

## Sample Request

```bash
curl -X POST https://YOUR-N8N-INSTANCE/webhook/appointment-request \
  -H "Content-Type: application/json" \
  -d '{
    "client_name": "Sarah Johnson",
    "client_email": "sarah@innovatetech.com",
    "requested_date": "2025-07-15",
    "requested_time": "10:00",
    "appointment_type": "demo",
    "reason": "We want to see a live demo of your enterprise automation platform before making a procurement decision.",
    "preferred_channel": "video"
  }'
```

---

## Sample Response — Routine Booking

```json
{
  "success": true,
  "priority_flag": false,
  "appointment": {
    "urgency": "routine",
    "classification": "demo",
    "validated_date": "2025-07-15",
    "validated_time": "10:00",
    "is_business_hours": true,
    "suggested_slot": "2025-07-15 at 10:00",
    "appointment_duration_minutes": 60,
    "channel_recommendation": "video",
    "preparation_notes": "Prepare a live demo environment focused on enterprise workflow automation features. Have case studies from similar procurement clients ready to share.",
    "client_confirmation_message": "Thank you for reaching out, Sarah! We are excited to show you our enterprise automation platform in action. We will confirm your demo slot for July 15th at 10:00 AM and send a calendar invite with the video link shortly."
  }
}
```

## Sample Response — Priority Booking (Urgent)

```json
{
  "success": true,
  "priority_flag": true,
  "appointment": {
    "urgency": "urgent",
    "classification": "support",
    "validated_date": "2025-06-11",
    "validated_time": "14:00",
    "is_business_hours": true,
    "suggested_slot": "2025-06-11 at 14:00",
    "appointment_duration_minutes": 45,
    "channel_recommendation": "video",
    "preparation_notes": "Client is experiencing a production issue. Pull up their account and recent support history before joining the call.",
    "client_confirmation_message": "We understand this is time-sensitive and we are prioritizing your request immediately. A specialist will join you at 14:00 today — please watch for the calendar invite in the next few minutes."
  }
}
```

---

## Input Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `client_name` | string | Yes | Full name of the person requesting the appointment |
| `client_email` | string | Yes | Email address for calendar invite and confirmation |
| `requested_date` | string | Yes | Preferred date (any format — AI normalizes to YYYY-MM-DD) |
| `requested_time` | string | Yes | Preferred time (any format — AI normalizes to HH:MM) |
| `appointment_type` | string | No | Type hint: demo, consultation, support, onboarding, etc. |
| `reason` | string | No | Why the client wants the appointment |
| `preferred_channel` | string | No | video, phone, or in_person |

---

## Output Fields

| Field | Type | Allowed Values | Description |
|---|---|---|---|
| `urgency` | string | routine, urgent, emergency | AI-assessed time sensitivity |
| `classification` | string | consultation, follow_up, demo, support, onboarding, general | Meeting type |
| `validated_date` | string or null | YYYY-MM-DD | Parsed and normalized date |
| `validated_time` | string or null | HH:MM | Parsed and normalized time |
| `is_business_hours` | boolean | true, false | Whether the slot falls within business hours |
| `suggested_slot` | string | — | Recommended slot (alternative if out of hours) |
| `appointment_duration_minutes` | integer | — | Estimated meeting length |
| `channel_recommendation` | string | video, phone, in_person | Recommended meeting format |
| `preparation_notes` | string | — | Host briefing before the meeting |
| `client_confirmation_message` | string | — | Ready-to-send client message |

---

## Priority Routing

The IF node routes on `urgency != "routine"`:

| Urgency | Branch | `priority_flag` | HTTP Header |
|---|---|---|---|
| `emergency` | Priority Booking | `true` | `X-Appointment-Priority: high` |
| `urgent` | Priority Booking | `true` | `X-Appointment-Priority: high` |
| `routine` | Routine Booking | `false` | `X-Appointment-Priority: normal` |

Urgency is determined by the AI based on:
- **emergency** — reason contains words like urgent, critical, emergency, ASAP
- **urgent** — requested date is today or tomorrow
- **routine** — all other requests

---

## Technologies Used

| Technology | Role |
|---|---|
| n8n (v2.27+) | Workflow orchestration |
| OpenAI GPT-4.1-mini | Date validation, urgency detection, slot suggestion |
| Webhook Node | HTTP intake for appointment requests |
| Set Node | Input normalization and auto-timestamping |
| Code Node | Safe JSON parsing and field validation |
| IF Node | Priority routing (urgent vs. routine) |
| Respond to Webhook | Structured HTTP response (two branches) |

---

## Troubleshooting

**Webhook 404** — Toggle workflow to Active. Use the Production URL (no `/test/` segment).

**OpenAI auth error** — Re-enter the API key. Confirm `gpt-4.1-mini` is available on your plan; substitute `gpt-4o-mini` if needed.

**`validated_date` or `validated_time` is null** — The AI could not parse the date/time from the input. Send dates in `YYYY-MM-DD` format and times in `HH:MM` for guaranteed parsing.

**Both branches return the same shape** — This is by design. `priority_flag` and the `X-Appointment-Priority` header are the routing signals for downstream systems.

**Code node returns defaults** — The execution log will show the raw OpenAI response. If it contains markdown fences, the Code node strips them automatically. Check for any other unexpected wrapper.

---

## License

MIT — free to use, modify, and deploy in commercial projects.
