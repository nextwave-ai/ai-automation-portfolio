# AI RAG Knowledge Base & Business Agent

⚙️ Runnable Workflow

An AI agent that answers business questions using company documents (RAG), and takes real action — booking meetings — when the user's intent requires it, with automatic human escalation when confidence is low.

## Why this is not "just a chatbot"

Most RAG demos stop at question → answer. This one closes the loop:

- **Question → Answer** grounded in retrieved documents, with source attribution
- **Question → Action** — if the user asks to book a meeting, the agent extracts intent, resolves a real datetime, and creates a Google Calendar event
- **Low confidence → Human handoff** — if no retrieved document is a good enough match, the question is escalated to a Slack channel instead of letting the AI guess

## Architecture

This is built as two independent workflows, because ingestion and querying have different triggers, frequencies, and failure modes.

### 1. Document Ingestion Pipeline

Document Text (via webhook) → Chunk Text (800 chars, 100 char overlap) → Generate Embedding (OpenAI text-embedding-3-small) → Prepare Row → Append to Google Sheet (vector store) → Respond Success

### 2. Query & Action Agent

Question (via webhook) → Embed Question → Read Knowledge Base (Google Sheets) → Semantic Search (cosine similarity, top 3 matches) → Confidence Check
- [true: score ≥ threshold] → Generate Answer (LLM, grounded in context) → Parse & Validate LLM Output → Intent Router
  - [true: book_meeting] → Book Meeting (Google Calendar) → Format Response → Respond With Answer
  - [false: answer] → Format Response → Respond With Answer
- [false: score < threshold] → Notify Human (Slack) → Respond Escalated

## Why Google Sheets as a vector store

No dedicated vector database (Pinecone, Weaviate, etc.) is used. Instead, embeddings are stored as JSON arrays in a Google Sheet, and retrieval is done with real cosine similarity computed in a Code node. This keeps the stack consistent with the rest of the portfolio (no new paid service, fully inspectable, fully runnable) while still being genuine semantic search — not keyword matching.

This is a deliberate tradeoff for small-to-medium knowledge bases (roughly under a few thousand chunks). For larger scale, the Read Knowledge Base + Semantic Search nodes are the only parts that would need to be swapped for a dedicated vector DB — the rest of the architecture is unchanged.

## Key engineering decisions

- **AI output is never trusted for dates/times directly.** The first version of this agent had the LLM invent meeting times from its training data (e.g. returning a 2023 date for "tomorrow at 2pm"). Fixed by (1) passing the current real datetime into the system prompt, and (2) validating the returned `meeting_time` in code — if it isn't a real, parseable, future date, the agent falls back to an "answer" response and flags the meeting for manual follow-up instead of writing garbage into Google Calendar.
- **Confidence threshold gates every answer.** The agent never generates an answer from a weak match. Below the similarity threshold, the question is escalated to Slack with the original question and its match score, instead of letting the model hallucinate from thin context.
- **LLM output is parsed defensively.** JSON responses are stripped of markdown fences and wrapped in try/catch; invalid `intent` values are coerced to a safe default rather than breaking the workflow.

## Setup to run this yourself

1. Create a Google Sheet named `KnowledgeBase` (or any name) with columns: `chunk_text | source | doc_id | chunk_index | embedding | created_at`
2. Import both workflow JSON files into n8n
3. Connect credentials: OpenAI (embeddings + chat completions), Google Sheets, Google Calendar, Slack
4. Update the Google Sheet reference and Slack channel in both workflows
5. POST to `/webhook/rag-ingest` to add documents, `/webhook/rag-query` to ask questions

## Example

**Ingest:**
```json
{"text": "We offer a full refund within 30 days...", "source": "Refund Policy Doc", "doc_id": "doc-001"}
```

**Query:**
```json
{"question": "What is your refund policy?"}
```

**Response:**
```json
{"answer": "We offer a full refund within 30 days...", "sources": ["Refund Policy Doc"], "action_taken": "none"}
```

## Stack

n8n (self-hosted) · OpenAI API (embeddings + chat) · Google Sheets · Google Calendar · Slack

## Tested execution

See `/screenshots` for a successful end-to-end run of both workflows, including the low-confidence escalation path and the meeting-booking action path.
