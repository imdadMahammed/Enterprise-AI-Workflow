# AI Classification Prompt

This is the system prompt used by the OpenAI node (`OpenAI - Classify Request`) in the n8n workflow. It is intentionally constrained to return strict JSON so it can be parsed reliably and fed into the routing logic without manual handling.

## System Prompt

```
You are an enterprise request triage assistant. Classify the employee
request into one of: HR, Finance, IT, Procurement. Extract a short
summary, a priority (Low, Medium, High), and a one-line suggested
response. Respond ONLY in strict JSON with keys: category, summary,
priority, suggested_response.
```

## Why this design

- **Strict JSON output** removes the need for fragile regex or keyword matching downstream, and lets the n8n `Switch` node route deterministically on `category`.
- **Closed category set** (HR, Finance, IT, Procurement) keeps routing predictable. An open-ended category field would break the Switch node and require a fallback path for every unexpected value.
- **Priority field** drives the high-priority manager approval branch, so urgent requests (e.g. "laptop dead before a client call") get a Slack alert instead of waiting in a queue.
- **Low temperature (0.2)** is used to keep classification consistent across similar requests.

## Known failure modes and mitigations

| Failure | Mitigation |
|---|---|
| Model returns malformed JSON | `Parse AI Output` code node should be wrapped in try/catch in production; failed parses route to a manual review queue instead of crashing the workflow |
| Ambiguous request (e.g. "I need help with my expense report system access") | Could match IT or Finance; current prompt does not handle multi-category cases — flagged as a known limitation, see README Future Improvements |
| Sensitive data in request body (salary, medical info) | Recommend redacting known sensitive fields before sending to the OpenAI API in any real deployment |
