# AI Appointment Scheduling Assistant

## Status

📄 This project was built and tested in n8n (see screenshots below). The exported `workflow.json` is not currently included in this repository, so it cannot be imported directly. The Google Docs and Email Delivery steps were part of the design but are marked optional — verify against the screenshots before assuming they were fully wired and tested end-to-end.


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

Client Request
↓
Input Validation
↓
OpenAI Proposal Generation
↓
Document Formatting
↓
Google Docs / PDF *(optional step — see Status below)*
↓
Email Delivery *(optional step — see Status below)*




