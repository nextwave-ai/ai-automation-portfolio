# AI WhatsApp Sales & Support Agent

⚙️ Runnable Workflow

An AI agent that receives WhatsApp messages via the Meta Cloud API, classifies whether the sender needs sales or support, answers accordingly, and sends the reply back to the customer on WhatsApp — with the support path grounded in a real company knowledge base rather than open-ended AI guessing.

## Why this is not "just a WhatsApp chatbot"

- **Real inbound/outbound integration** — not a simulated demo. Built and tested against Meta's actual WhatsApp Business Cloud API, including webhook verification, signature-style token checks, and live message delivery back to a real phone.
- **Intent-based routing** — every inbound message is classified (sales vs. support) before a reply is generated, so a pricing question and a policy question get fundamentally different handling, not the same generic AI response.
- **Support answers are grounded, not invented** — support questions are routed to the [AI RAG Knowledge Base & Business Agent](../AI-RAG-Knowledge-Base-Agent) project via its own webhook, reusing that project's document retrieval instead of duplicating logic. This is a deliberate architecture choice: one knowledge base, multiple channels (WhatsApp here, direct API in the RAG project).
- **Every conversation is logged** — inbound and outbound messages are both written to a Google Sheet conversation log, so nothing lives only inside WhatsApp.

## Architecture

Meta WhatsApp Cloud API → Webhook (GET: verification / POST: messages) → Extract Message → Log Inbound Message → Classify Intent (AI)
→ [support: Query Knowledge Base (calls the RAG project's webhook) → Extract Support Reply]
→ [sales: Generate Sales Reply (AI)] → Send WhatsApp Reply (Graph API) → Log Outbound Message

The webhook verification (GET) and message receipt (POST) share a single endpoint on two HTTP methods, matching how Meta's Cloud API is designed to call back a single Callback URL for both.

## Setup

1. Create a Meta Developer App with the "Connect with customers through WhatsApp" use case, claim a test number, and generate a permanent System User access token with `whatsapp_business_messaging` permission.
2. Expose your local n8n instance publicly (e.g. via Cloudflare Tunnel or ngrok) and register the resulting URL as the app's Webhook Callback URL, subscribed to the `messages` field.
3. Import `whatsapp-ai-agent-workflow.json` into n8n.
4. Create credentials: OpenAI (chat completions), Google Sheets, and a Header Auth credential named `Authorization` with value `Bearer <your permanent access token>` for the Send WhatsApp Reply node.
5. Create a Google Sheet with columns: `timestamp | from | contact_name | message_text | message_id | direction`.
6. For the support path to work, the [AI RAG Knowledge Base & Business Agent](../AI-RAG-Knowledge-Base-Agent)'s query workflow must also be imported, configured, and active in the same n8n instance, since this project calls it directly by webhook URL.

## Key engineering decisions

- **Immediate acknowledgment, background processing.** The incoming webhook responds to Meta immediately (`onReceived` mode) rather than waiting for the full AI pipeline to finish, since Meta expects a fast response and will retry or flag the endpoint as unhealthy otherwise.
- **Reused RAG infrastructure instead of a second knowledge base.** Rather than building a separate embeddings store for WhatsApp support questions, this project calls the existing RAG project's query webhook. One knowledge base to maintain, two channels served.
- **Verify token is a shared secret, not a credential.** Webhook verification uses a plain string comparison against a token only n8n and the Meta app configuration know — this is standard practice for Meta Cloud API webhooks and is separate from the OAuth access token used to send messages.

## Test coverage

- **Sales path: tested end-to-end with real message delivery.** A live WhatsApp message was sent to the test number, correctly classified as sales intent, answered by the AI, and delivered back to a real phone via the Graph API — confirmed by execution logs and by receiving the reply on WhatsApp.
- **Support path: logic implemented and integrated, partially tested.** The intent classification and the call to the RAG project's webhook are wired and were exercised during development; full confirmation of a delivered reply on this path requires both this workflow and the RAG project's query workflow to be active simultaneously in the same n8n instance. Anyone rebuilding this should verify that dependency is running before testing the support path.

## Stack

n8n (self-hosted) · Meta WhatsApp Business Cloud API · OpenAI API (gpt-4o-mini) · Google Sheets · Cloudflare Tunnel (for local webhook exposure during development)
