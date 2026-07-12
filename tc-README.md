# AI Customer Support Ticket Classifier
### n8n + OpenAI GPT-4.1-mini — Production Ready — v1.0

Automatically classifies inbound customer support requests by category, urgency, sentiment, and escalation need — in under 3 seconds, via a single webhook call.

---

## What This Workflow Does

1. Receives a customer message via HTTP POST (from any source: web widget, email parser, CRM, or direct API)
2. Normalizes the payload into a consistent internal schema
3. Sends the message to OpenAI GPT-4.1-mini with a structured classification prompt
4. Safely parses the AI response (with fallback protection if the response is malformed)
5. Returns a clean JSON classification to the caller

**Example output:**

```json
{
  "success": true,
  "classification": {
    "category": "billing",
    "urgency": "high",
    "sentiment": "negative",
    "summary": "Customer was charged twice for their subscription and is requesting a refund.",
    "needs_human": true,
    "suggested_action": "Escalate to billing team and initiate refund investigation."
  }
}
```

---

## Requirements

- n8n version **2.27 or higher** (cloud or self-hosted)
- OpenAI API account with access to **gpt-4.1-mini**
- An HTTP client for testing (curl, Postman, or similar)

---

## Installation

### Step 1 — Import the Workflow

1. Open your n8n instance
2. Go to **Workflows** in the left sidebar
3. Click **Add Workflow → Import from File**
4. Upload `workflow.json`
5. Click **Save**

### Step 2 — Add Your OpenAI Credential

1. In the imported workflow, click on the **"Classify with OpenAI"** node
2. In the **Credential** field, click **Create New**
3. Select type: **OpenAI**
4. Paste your OpenAI API Key (`sk-...`)
5. Click **Save**

### Step 3 — Activate the Workflow

1. Toggle the workflow to **Active** (top-right switch)
2. Click the **Webhook** node
3. Copy the **Production URL** — it will look like:

```
https://your-n8n-instance.com/webhook/customer-support
```

---

## Testing

### Quick Test with curl

Replace `YOUR-WEBHOOK-URL` with the URL from the Webhook node:

```bash
curl -X POST https://YOUR-WEBHOOK-URL/webhook/customer-support \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ali",
    "email": "ali@test.com",
    "message": "My order has not arrived. It has been 10 days and I need help immediately."
  }'
```

### Expected Response

```json
{
  "success": true,
  "classification": {
    "category": "general",
    "urgency": "high",
    "sentiment": "negative",
    "summary": "Customer's order has not arrived after 10 days and requires urgent attention.",
    "needs_human": true,
    "suggested_action": "Escalate to fulfillment team and check order tracking status."
  }
}
```

### More Test Payloads

See `test-payload.json` for 5 ready-to-use test cases covering all major ticket types.

---

## Workflow Architecture

```
POST /webhook/customer-support
        │
        ▼
[Webhook] — Receives { name, email, message }
        │
        ▼
[Normalize Input] — Maps to { customer_name, customer_email, customer_message }
        │
        ▼
[Classify with OpenAI] — GPT-4.1-mini classifies message → JSON string
        │
        ▼
[Parse Classification] — Safe JSON parse with field validation + fallback
        │
        ▼
[Respond to Webhook] — Returns { success: true, classification: { ... } }
```

---

## Configuration

### Changing the AI Model

In the **"Classify with OpenAI"** node, click the **Model** dropdown and select any available model. `gpt-4.1-mini` is recommended for cost-efficiency. Use `gpt-4o` for maximum accuracy.

### Adjusting Classification Categories

Edit the system prompt in the **"Classify with OpenAI"** node. Find this line:

```
"category": "one of: billing | technical | onboarding | account | general"
```

Add or change categories as needed. Also update the validation array in the **"Parse Classification"** Code node:

```javascript
const validCategories = ['billing', 'technical', 'onboarding', 'account', 'general'];
```

### Changing the Webhook Path

In the **Webhook** node, change the **Path** field from `customer-support` to any path you prefer. Your new URL will be:

```
https://your-n8n-instance.com/webhook/YOUR-NEW-PATH
```

---

## Troubleshooting

### "Webhook not found" / 404 Error

**Cause:** The workflow is not active, or you are using the test URL instead of the production URL.

**Fix:**
- Toggle the workflow to **Active** (the grey switch in the top-right should turn green)
- Use the **Production URL** from the Webhook node, not the **Test URL**
- The production URL does **not** contain `/test/` in the path

---

### OpenAI node shows "Authentication failed"

**Cause:** Missing or invalid OpenAI credential.

**Fix:**
- Click the OpenAI node → Credential field → verify the API key is correct
- Ensure your OpenAI account has an active billing method and available credits
- Confirm `gpt-4.1-mini` is available in your OpenAI account tier

---

### OpenAI node shows "Model not found"

**Cause:** `gpt-4.1-mini` may not be available on your OpenAI account tier.

**Fix:** In the OpenAI node, change the model to `gpt-4o-mini` (the previous generation equivalent). The system prompt and all other configuration remains identical.

---

### Response returns the fallback classification

**Cause:** The Code node triggered the fallback because OpenAI returned an unexpected format.

**Symptom:** Response contains `"summary": "Classification failed. Manual review required."`

**Fix:**
- Open the n8n execution log for the failed run
- Click the **"Classify with OpenAI"** node in the execution to see the raw response
- If OpenAI returned markdown-wrapped JSON (```json ... ```), the Code node strips it automatically — check for any other wrapper format
- Lower the temperature setting in the OpenAI node (it is already 0.1, the minimum useful value)

---

### Set node outputs empty fields

**Cause:** The incoming webhook payload uses different field names than expected.

**Fix:** The workflow expects the body to contain `name`, `email`, and `message`. If your caller sends different field names (e.g., `customer_name`, `text`), update the expressions in the **Normalize Input** node:

```
customer_name  →  {{ $json.body.YOUR_FIELD_NAME }}
customer_email →  {{ $json.body.YOUR_FIELD_NAME }}
customer_message → {{ $json.body.YOUR_FIELD_NAME }}
```

---

### n8n version compatibility

This workflow requires **n8n 2.27 or higher**.

Node versions used:
- Webhook: typeVersion 2
- Set: typeVersion 3.4
- OpenAI: typeVersion 1.8
- Code: typeVersion 2
- Respond to Webhook: typeVersion 1.1

If you are on an older version of n8n, upgrade via:
```bash
# Docker
docker pull docker.n8n.io/n8nio/n8n:latest

# npm
npm update -g n8n
```

---

## License

MIT — free to use, modify, and deploy in commercial projects.
