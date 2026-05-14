# Webhook Examples

This document explains how webhooks work in n8n and provides ready-to-use curl examples for each workflow trigger in this kit.

---

## What Are Webhooks in n8n?

A webhook is an HTTP endpoint that n8n exposes and listens on. When an external system (a form, CRM, app, or script) sends an HTTP request to that URL, n8n receives the payload and starts executing the workflow.

The **Webhook** node in n8n is the trigger. It supports:

- HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`
- Authentication: None, Basic Auth, Header Auth
- Response modes: Immediately (acknowledge receipt) or after the workflow finishes

---

## Test Mode vs Production Mode

n8n provides two distinct webhook URLs for every Webhook node.

| Mode | URL Format | When It Works |
|------|-----------|---------------|
| Test | `/webhook-test/{path}` | Only while the workflow editor is open and listening |
| Production | `/webhook/{path}` | Only after the workflow is **Activated** |

### Test URL

Use the test URL during development. n8n shows it in the Webhook node panel when you open it in the editor.

```
http://localhost:5678/webhook-test/lead-submission
```

To start listening, click the Webhook node and click **Listen for Test Event**, then send your request.

### Production URL

After you activate the workflow (toggle the switch to Active), use the production URL in your actual integrations:

```
http://localhost:5678/webhook/lead-submission
```

**Never use the test URL in production systems.** It stops working as soon as you close the editor tab.

---

## Webhook URL Format

```
{PROTOCOL}://{HOST}:{PORT}/webhook/{your-path}
```

If you set `WEBHOOK_URL` in your `.env`, n8n uses that as the base:

```
https://your-domain.com/webhook/{your-path}
```

The `{your-path}` is set in the Webhook node's **Path** field. It must be unique across all active workflows.

---

## Workflow 1: Lead Submission Webhook

**Workflow file:** `workflows/lead-capture-ai-qualifier.json`
**Webhook path:** `lead-submission`
**Method:** POST

This webhook receives a new lead from a web form or CRM and runs AI qualification.

### curl Example

```bash
curl -X POST http://localhost:5678/webhook/lead-submission \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_secret_token" \
  -d '{
    "name": "Jane Smith",
    "email": "jane.smith@acmecorp.com",
    "company": "Acme Corp",
    "title": "VP of Engineering",
    "employees": 250,
    "budget": 50000,
    "message": "We need a solution to automate our onboarding process for enterprise clients. Looking for something we can deploy this quarter.",
    "source": "website-contact-form"
  }'
```

### Sample Payload Schema

```json
{
  "name": "string — full name of the lead",
  "email": "string — work email address",
  "company": "string — company name",
  "title": "string — job title",
  "employees": "number — company size",
  "budget": "number — estimated budget in USD",
  "message": "string — their inquiry or message",
  "source": "string — where the lead came from"
}
```

### Expected Response

```json
{
  "status": "received",
  "message": "Lead submitted successfully. AI qualification in progress."
}
```

---

## Workflow 2: Support Ticket Webhook

**Workflow file:** `workflows/support-ticket-classifier.json`
**Webhook path:** `support-ticket`
**Method:** POST

This webhook receives a new support request and routes it to the appropriate team based on AI classification.

### curl Example

```bash
curl -X POST http://localhost:5678/webhook/support-ticket \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_secret_token" \
  -d '{
    "ticket_id": "TKT-20240514-001",
    "customer_name": "Robert Johnson",
    "customer_email": "robert@example.com",
    "customer_plan": "enterprise",
    "subject": "Cannot export reports to PDF",
    "body": "Hi, I have been trying to export my monthly reports to PDF format for the past two days. The export button is greyed out and nothing happens when I click it. I have tried in Chrome and Firefox. This is urgent as I need these reports for a board meeting tomorrow morning.",
    "created_at": "2024-05-14T09:30:00Z",
    "priority": "high"
  }'
```

### Sample Payload Schema

```json
{
  "ticket_id": "string — unique ticket identifier",
  "customer_name": "string",
  "customer_email": "string",
  "customer_plan": "string — free | pro | enterprise",
  "subject": "string — ticket subject line",
  "body": "string — full description of the issue",
  "created_at": "string — ISO 8601 timestamp",
  "priority": "string — low | medium | high | urgent"
}
```

### Expected Response

```json
{
  "status": "received",
  "ticket_id": "TKT-20240514-001",
  "message": "Ticket received. Classification in progress."
}
```

---

## Workflow 3: Meeting Transcript Webhook

**Workflow file:** `workflows/meeting-notes-generator.json`
**Webhook path:** `meeting-transcript`
**Method:** POST

This webhook accepts a raw meeting transcript and generates structured notes, action items, and decisions.

### curl Example

```bash
curl -X POST http://localhost:5678/webhook/meeting-transcript \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_secret_token" \
  -d '{
    "meeting_id": "mtg-2024-05-14-001",
    "title": "Q2 Product Roadmap Review",
    "date": "2024-05-14",
    "duration_minutes": 45,
    "attendees": ["Alice Chen", "Bob Kumar", "Carol White", "David Park"],
    "transcript": "Alice: Good morning everyone. Lets start with the roadmap review. Bob, can you walk us through the current sprint status? Bob: Sure. We finished the API redesign last week and are now 80% done with the dashboard refresh. We should ship by end of next week. Alice: Great. Any blockers? Bob: The main blocker is the design review from Carol. Carol: Yes, I am reviewing it today. I will have feedback by end of day. Alice: Perfect. David, what about the mobile app timeline? David: We are targeting a beta release on June 1st. The login flow is done. Push notifications are in progress. Alice: OK. Action item: Carol to send design feedback by EOD today. David to share beta build by May 28th for testing. We will reconvene next Tuesday at 10am. Does everyone have the invite? All: Yes. Alice: Great, thanks everyone.",
    "source": "zoom-integration"
  }'
```

### Sample Payload Schema

```json
{
  "meeting_id": "string — unique identifier",
  "title": "string — meeting name",
  "date": "string — YYYY-MM-DD",
  "duration_minutes": "number",
  "attendees": "array of strings — participant names",
  "transcript": "string — raw transcript text",
  "source": "string — zoom | teams | google-meet | manual"
}
```

### Expected Response

```json
{
  "status": "received",
  "meeting_id": "mtg-2024-05-14-001",
  "message": "Transcript received. Notes generation in progress."
}
```

---

## Workflow 4: Invoice Text Webhook

**Workflow file:** `workflows/invoice-extractor.json`
**Webhook path:** `invoice-extract`
**Method:** POST

This webhook receives raw invoice text (pasted or OCR'd) and extracts structured data fields.

### curl Example

```bash
curl -X POST http://localhost:5678/webhook/invoice-extract \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_secret_token" \
  -d '{
    "invoice_id": "INV-RAW-001",
    "raw_text": "INVOICE\n\nFrom: TechSupplies Ltd\n123 Business Park, London, UK\nVAT Number: GB123456789\n\nTo: Acme Corporation\n456 Main Street, New York, NY 10001\n\nInvoice Number: TS-2024-0892\nInvoice Date: May 10, 2024\nDue Date: June 10, 2024\n\nDescription                    Qty    Unit Price    Total\nLaptop Stand - Ergonomic        5      £45.00       £225.00\nUSB-C Hub 7-port                10     £29.99       £299.90\nWireless Keyboard               3      £89.00       £267.00\n\nSubtotal: £791.90\nVAT (20%): £158.38\nTotal Due: £950.28\n\nPayment Terms: Net 30\nBank: Barclays | Sort: 20-00-00 | Account: 12345678",
    "source": "email-attachment",
    "submitted_by": "accounts@acmecorp.com"
  }'
```

### Sample Payload Schema

```json
{
  "invoice_id": "string — your internal reference",
  "raw_text": "string — full invoice text, may include newlines",
  "source": "string — email-attachment | ocr-scan | manual-paste",
  "submitted_by": "string — email of the person submitting"
}
```

### Expected Response

```json
{
  "status": "received",
  "invoice_id": "INV-RAW-001",
  "message": "Invoice text received. Extraction in progress."
}
```

---

## Authentication Headers Reference

### No Authentication (development only)

```bash
curl -X POST http://localhost:5678/webhook-test/your-path \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### Basic Authentication

```bash
# Encode credentials: username:password in base64
CREDENTIALS=$(echo -n "webhook_user:your_password" | base64)

curl -X POST https://your-domain.com/webhook/your-path \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic $CREDENTIALS" \
  -d '{"key": "value"}'
```

### Header Authentication (Recommended)

```bash
curl -X POST https://your-domain.com/webhook/your-path \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: your_32_char_random_secret" \
  -d '{"key": "value"}'
```

### Testing All Workflows at Once (Shell Script)

```bash
#!/bin/bash
BASE_URL="http://localhost:5678/webhook-test"
SECRET="your_secret_token"

echo "Testing lead submission..."
curl -s -X POST "$BASE_URL/lead-submission" \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: $SECRET" \
  -d '{"name":"Test Lead","email":"test@example.com","company":"Test Co","message":"Need help"}' | jq .

echo "Testing support ticket..."
curl -s -X POST "$BASE_URL/support-ticket" \
  -H "Content-Type: application/json" \
  -H "X-Webhook-Secret: $SECRET" \
  -d '{"ticket_id":"TKT-001","customer_name":"Test User","customer_email":"user@test.com","subject":"Test issue","body":"This is a test ticket","priority":"medium"}' | jq .

echo "Done."
```

---

## Troubleshooting Webhooks

| Problem | Likely Cause | Solution |
|---------|-------------|---------|
| `404 Not Found` | Wrong URL path or workflow not activated | Check path spelling; activate the workflow for production URLs |
| `401 Unauthorized` | Missing or wrong auth header | Add the correct `Authorization` or `X-Webhook-Secret` header |
| Workflow not triggering | Using production URL while workflow is inactive | Activate the workflow or use the test URL |
| Workflow triggers but fails immediately | Payload field names don't match expressions | Check the first node's expected field names against your payload |
| Timeout waiting for response | Workflow set to respond "After Last Node" and is slow | Set response mode to "Immediately" for long-running workflows |
