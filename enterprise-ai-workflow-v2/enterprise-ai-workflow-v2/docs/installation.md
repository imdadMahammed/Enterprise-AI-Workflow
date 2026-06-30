# Installation & Setup Guide

This guide walks through deploying this workflow on a real n8n instance.

## Prerequisites

- An n8n instance (self-hosted via Docker, or n8n Cloud)
- An OpenAI API key
- Credentials for whichever target systems you connect (Jira, Gmail, Google Sheets, Slack)

## 1. Run n8n locally with Docker

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Then open `http://localhost:5678` in your browser.

## 2. Import the workflow

1. In n8n, click **Workflows** → **Import from File**
2. Select `workflows/employee-request-automation.json` from this repository
3. The full node graph (Webhook → OpenAI → Switch → integrations → approval → audit log) will load automatically

## 3. Add credentials

In each node (OpenAI, Jira, Gmail, Google Sheets, Slack), click the node and attach your own API credentials. None are stored in this repository.

## 4. Set environment-specific values

Replace placeholder fields before activating:
- `itProjectKey`: your Jira project key
- `financeSheetId` / `auditLogSheetId`: your Google Sheet IDs
- Email addresses in the Gmail nodes
- Slack channel name

## 5. Activate and test

Use the sample payloads in `examples/` to send test requests to the webhook URL n8n generates, using a tool like `curl` or Postman.

```bash
curl -X POST https://your-n8n-instance/webhook/employee-request \
  -H "Content-Type: application/json" \
  -d @examples/sample-it-request.json
```

## Security notes

- API keys should always be stored in n8n's credential manager, never hardcoded in the workflow JSON
- Consider redacting sensitive fields (salary, medical details) before they reach the OpenAI API
- Restrict the webhook endpoint with an auth header or IP allowlist in production
