# AI Workflow Automation Kit for n8n

**OpenAI-Powered Business Workflow Templates**

A production-ready library of n8n workflow templates powered by OpenAI for common business operations. Clone, configure, import into n8n, and start automating.

---

## Who This Is For

- Business teams looking to automate repetitive tasks with AI
- n8n users who want ready-made OpenAI-powered workflows
- Developers building AI-driven automation pipelines
- Agencies delivering automation solutions to clients

## Workflow Templates

| # | Workflow | File | Key Features |
|---|---------|------|--------------|
| 1 | **AI Email Summarizer** | `workflows/01-ai-email-summarizer.json` | Gmail trigger, 3-bullet summary, urgency detection, Slack alerts |
| 2 | **AI Lead Qualification & Follow-Up** | `workflows/02-ai-lead-qualification-and-follow-up.json` | Lead scoring 0-100, personalized follow-up, human approval |
| 3 | **Support Ticket Classifier** | `workflows/03-support-ticket-classifier.json` | Category/priority/sentiment classification, routing, retry logic |
| 4 | **Meeting Notes Generator** | `workflows/04-meeting-notes-generator.json` | Transcript to structured notes, action items, Notion integration |
| 5 | **Google Sheets CRM Enrichment** | `workflows/05-google-sheets-crm-enrichment.json` | AI-powered record classification, CRM sync, priority alerts |
| 6 | **Invoice Data Extraction** | `workflows/06-invoice-data-extraction.json` | Structured extraction, line items, human approval for high-value |
| 7 | **Daily Business Report** | `workflows/07-daily-business-report-generator.json` | Cron-triggered, multi-source data, executive summary |
| 8 | **Human Approval AI Response** | `workflows/08-human-approval-ai-response.json` | Reusable human-in-the-loop pattern, Slack approval flow |

## Required Tools

- [n8n](https://n8n.io/) (self-hosted or cloud)
- [OpenAI API key](https://platform.openai.com/api-keys)
- Optional: Gmail, Google Sheets, Slack, Notion, CRM accounts

## Quick Start

### Option 1: Docker (Recommended)

```bash
# Clone the repo
git clone https://github.com/humayun-sarfraz/n8n-ai-workflow-kit.git
cd n8n-ai-workflow-kit

# Configure environment
cp .env.example .env
# Edit .env with your credentials

# Start n8n
docker compose up -d

# Open n8n at http://localhost:5678
```

### Option 2: npm

```bash
# Install n8n globally
npm install -g n8n

# Start n8n
n8n start

# Open n8n at http://localhost:5678
```

## Configure OpenAI Credentials

1. Open n8n UI at `http://localhost:5678`
2. Go to **Settings > Credentials**
3. Click **Add Credential**
4. Select **OpenAI API**
5. Paste your API key
6. Click **Save**

See [docs/credentials-guide.md](docs/credentials-guide.md) for all credential types.

## Import a Workflow

1. Open n8n UI
2. Click **Workflows > Import from File**
3. Select a JSON file from the `workflows/` folder
4. Assign your credentials to each node that requires them
5. Click **Test Workflow** to verify
6. Toggle **Active** to enable

See [docs/workflow-import-guide.md](docs/workflow-import-guide.md) for detailed steps.

## Testing Workflows

### Webhook-based workflows

Use `curl` to send test payloads:

```bash
# Test Lead Qualification
curl -X POST http://localhost:5678/webhook/lead-qualification \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/lead-submission.json

# Test Support Ticket Classifier
curl -X POST http://localhost:5678/webhook/support-ticket \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/support-ticket.json

# Test Meeting Notes
curl -X POST http://localhost:5678/webhook/meeting-notes \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/meeting-transcript.json

# Test Invoice Extraction
curl -X POST http://localhost:5678/webhook/invoice-extraction \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/invoice-text.json
```

### Trigger-based workflows

- **Email Summarizer**: Send a test email to the connected Gmail account
- **CRM Enrichment**: Add a new row to the connected Google Sheet
- **Daily Report**: Manually execute or wait for the 9 AM cron trigger

See [docs/webhook-examples.md](docs/webhook-examples.md) for full payload examples.

## Project Structure

```
n8n-ai-workflow-kit/
  README.md
  .env.example
  docker-compose.yml
  workflows/           # 8 ready-to-import n8n workflow JSON templates
  docs/                # Setup, credentials, security, troubleshooting guides
  examples/
    webhook-payloads/  # Sample JSON payloads for testing webhooks
    sample-outputs/    # Example AI-generated outputs
  prompts/             # Reusable OpenAI prompt templates
```

## Documentation

| Document | Description |
|----------|-------------|
| [Setup Guide](docs/setup-guide.md) | Install and configure n8n locally |
| [Credentials Guide](docs/credentials-guide.md) | Configure all API credentials |
| [Workflow Import Guide](docs/workflow-import-guide.md) | Step-by-step import instructions |
| [Webhook Examples](docs/webhook-examples.md) | Webhook URLs, payloads, and curl commands |
| [Prompt Library](docs/prompt-library.md) | Reusable AI prompts for all workflows |
| [Error Handling Patterns](docs/error-handling-patterns.md) | Retry logic, error alerts, validation |
| [Security Best Practices](docs/security-best-practices.md) | API key safety, data sanitization |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and solutions |

## Security Notes

- **Never hardcode API keys** — use n8n's built-in credential system
- **Use webhook authentication** — enable Basic Auth or Header Auth on webhook nodes
- **Sanitize inputs** — validate and clean webhook data before processing
- **Limit PII sent to OpenAI** — strip unnecessary personal data
- **Human approval** — use approval workflows for customer-facing AI responses
- **Rotate credentials** regularly

See [docs/security-best-practices.md](docs/security-best-practices.md) for details.

## Customization

Each workflow is designed to be adapted:

- **Change the AI model**: Update the model parameter in OpenAI nodes (e.g., `gpt-4o`, `gpt-4o-mini`)
- **Modify prompts**: Edit the system/user messages in OpenAI nodes or reference `prompts/` files
- **Swap integrations**: Replace Slack with Teams, Gmail with Outlook, Google Sheets with Airtable
- **Add nodes**: Extend workflows with additional processing, storage, or notification steps
- **Adjust thresholds**: Change lead score cutoffs, invoice amount thresholds, urgency rules

## Troubleshooting

| Issue | Solution |
|-------|----------|
| OpenAI credential error | Verify API key in n8n credentials, check billing status |
| Gmail trigger not firing | Ensure OAuth2 is configured, check Gmail API is enabled |
| Webhook not receiving data | Use n8n test mode first, check URL matches exactly |
| JSON parse error from AI | Add error handling branch, validate AI output format |
| Rate limit (429) errors | Enable retry logic on OpenAI nodes, add delay between calls |

See [docs/troubleshooting.md](docs/troubleshooting.md) for the full guide.

## License

MIT License. Use these templates freely in personal and commercial projects.

---

Built for the n8n community. Contributions welcome.
