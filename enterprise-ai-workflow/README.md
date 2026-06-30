# Enterprise AI Workflow Automation Platform

An AI-powered workflow automation system that classifies and routes employee requests (IT, HR, Finance, Procurement) automatically using OpenAI and n8n, with priority-based manager escalation and audit logging.

![Architecture Diagram](architecture/architecture-diagram.svg)

## The Problem

Large organizations receive a constant stream of repetitive employee requests across HR, Finance, IT, and Procurement. Most of these requests follow predictable patterns but still require manual triage: someone has to read the message, figure out which team owns it, decide how urgent it is, and route it to the right queue. That manual step is slow, inconsistent, and doesn't scale.

## What This Project Does

This workflow takes a request from Slack, email, or a web form and automatically:

1. **Classifies** it into a department (IT, HR, Finance, Procurement) using OpenAI
2. **Extracts** a summary, priority level, and a suggested response
3. **Routes** it to the right destination — a Jira ticket for IT, an email to HR, a row in a Finance tracking sheet, or an email to Procurement
4. **Escalates** high-priority requests to a manager via Slack for approval
5. **Logs** every decision to an audit trail for compliance and review

## Architecture

```
Employee Request (Slack / Email / Web Form)
        │
        ▼
  n8n Webhook Trigger
        │
        ▼
  OpenAI Classification
  (category, priority, summary, suggested response)
        │
        ▼
  Route by Department (Switch)
   ├── IT          → Jira Ticket
   ├── HR          → Email HR Team
   ├── Finance     → Google Sheets Log
   └── Procurement → Email Procurement
        │
        ▼
  If High Priority → Slack Alert to Manager
        │
        ▼
  Audit Log (Google Sheets)
```

See the full diagram in [`architecture/architecture-diagram.svg`](architecture/architecture-diagram.svg).

## Technologies Used

| Category | Tools |
|---|---|
| Workflow Engine | n8n |
| AI / NLP | OpenAI API (GPT-4o-mini) |
| Integrations | Slack, Gmail, Google Sheets, Jira |
| Data Format | JSON, REST Webhooks |
| Deployment | Docker |

## Repository Structure

```
enterprise-ai-workflow/
├── README.md
├── architecture/
│   └── architecture-diagram.svg
├── workflows/
│   └── employee-request-automation.json   ← importable n8n workflow
├── prompts/
│   └── classification-prompt.md           ← AI prompt design + rationale
├── docs/
│   └── installation.md                    ← setup guide
├── examples/
│   ├── sample-it-request.json
│   └── sample-finance-request.json
└── LICENSE
```

## Example

**Input** (employee message via Slack):
> "My laptop won't turn on and I have a client call in an hour. Please help urgently."

**AI Output:**
```json
{
  "category": "IT",
  "summary": "Laptop not powering on, urgent due to upcoming client call",
  "priority": "High",
  "suggested_response": "We've created an urgent IT ticket and a technician will reach out within 15 minutes."
}
```

**Resulting automation:**
- A Jira ticket is created in the IT project with High priority
- Because priority is High, a Slack alert is sent to the manager approval channel
- The decision is written to the audit log with a timestamp

More examples in [`examples/`](examples/).

## Design Decisions

- **Strict JSON output from the AI model** — the classification prompt forces a fixed schema so the routing logic downstream is deterministic, not based on fragile keyword matching. Full reasoning in [`prompts/classification-prompt.md`](prompts/classification-prompt.md).
- **Closed category set** — keeping categories to IT/HR/Finance/Procurement avoids unpredictable routing paths; ambiguous requests are a known limitation (see below).
- **Audit logging on every request** — not just high-priority ones — because access and approval decisions in an enterprise setting need to be traceable, not just automated.

## Setup

Full instructions in [`docs/installation.md`](docs/installation.md). Short version: run n8n via Docker, import the workflow JSON, attach your own API credentials, and test with the sample payloads in `examples/`.

## Known Limitations / Future Improvements

- No handling yet for requests that span multiple departments (e.g. "I need a new laptop approved by Finance")
- No retry/backoff logic if the OpenAI API call fails — a production version should queue and retry
- Approval workflow is currently single-manager; could be extended to multi-level approval chains
- No rate limiting on the webhook endpoint
- Planned: migrate classification to a LangGraph agent for multi-step reasoning on ambiguous requests
- Planned: add a small dashboard (Google Sheets-backed or a lightweight web app) to visualize request volume and turnaround time by department

## Author

Built by Imdad as a portfolio project demonstrating enterprise AI workflow automation, drawing on a background in Identity & Access Management — particularly in the audit logging and approval-routing design choices above.
