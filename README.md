<p align="center">
  <img src="https://img.shields.io/badge/n8n-Workflow%20Automation-FF6D5A?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai&logoColor=white" alt="OpenAI" />
  <img src="https://img.shields.io/badge/Templates-8%20Workflows-00C853?style=for-the-badge" alt="8 Workflows" />
</p>

<h1 align="center">AI Workflow Automation Kit for n8n</h1>

<p align="center">
  <strong>Production-ready OpenAI-powered business workflow templates</strong><br/>
  Clone. Configure. Import. Automate.
</p>

<p align="center">
  <a href="https://github.com/humayun-sarfraz/n8n-ai-workflow-kit/stargazers"><img src="https://img.shields.io/github/stars/humayun-sarfraz/n8n-ai-workflow-kit?style=social" alt="Stars" /></a>
  <a href="https://github.com/humayun-sarfraz/n8n-ai-workflow-kit/blob/main/LICENSE"><img src="https://img.shields.io/github/license/humayun-sarfraz/n8n-ai-workflow-kit?style=flat-square" alt="License" /></a>
  <a href="https://github.com/humayun-sarfraz/n8n-ai-workflow-kit/issues"><img src="https://img.shields.io/github/issues/humayun-sarfraz/n8n-ai-workflow-kit?style=flat-square" alt="Issues" /></a>
</p>

---

## Overview

A reusable library of **8 production-ready n8n workflow templates** powered by OpenAI for common business operations. Each workflow includes structured AI prompts, error handling, conditional logic, and integration with popular tools like Gmail, Slack, Google Sheets, Notion, and CRM APIs.

> **No backend required.** Import the JSON templates directly into n8n, configure your credentials, and go.

---

## Who This Is For

| Audience | Use Case |
|----------|----------|
| **Business Teams** | Automate repetitive tasks with AI — email triage, lead scoring, reporting |
| **n8n Users** | Drop-in workflow templates ready to customize |
| **Developers** | Reference implementations for AI-driven automation pipelines |
| **Agencies** | White-label automation solutions for clients |

---

## Workflow Templates

| # | Workflow | Description | Key Features |
|:-:|---------|-------------|:------------:|
| 1 | **AI Email Summarizer** | Summarize incoming emails with urgency detection | Gmail, Slack, Sheets |
| 2 | **Lead Qualification & Follow-Up** | Score leads 0-100 and generate personalized follow-ups | Webhook, CRM, Human Approval |
| 3 | **Support Ticket Classifier** | Classify tickets by category, priority, and sentiment | Webhook, Slack, Notion, Retry Logic |
| 4 | **Meeting Notes Generator** | Turn transcripts into structured notes and action items | Webhook, Notion, Human Review |
| 5 | **Google Sheets CRM Enrichment** | Enrich records with AI classification | Sheets Trigger, CRM API |
| 6 | **Invoice Data Extraction** | Extract structured data from invoice text | Webhook, Human Approval, Validation |
| 7 | **Daily Business Report** | Generate daily executive summaries from multiple sources | Cron Trigger, Multi-source |
| 8 | **Human Approval AI Response** | Reusable approve/reject/edit pattern for AI responses | Slack Approval, Gmail |

---

## Tech Stack

<p>
  <img src="https://img.shields.io/badge/n8n-FF6D5A?style=flat-square&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/OpenAI-412991?style=flat-square&logo=openai&logoColor=white" alt="OpenAI" />
  <img src="https://img.shields.io/badge/Gmail-EA4335?style=flat-square&logo=gmail&logoColor=white" alt="Gmail" />
  <img src="https://img.shields.io/badge/Google%20Sheets-34A853?style=flat-square&logo=googlesheets&logoColor=white" alt="Google Sheets" />
  <img src="https://img.shields.io/badge/Slack-4A154B?style=flat-square&logo=slack&logoColor=white" alt="Slack" />
  <img src="https://img.shields.io/badge/Notion-000000?style=flat-square&logo=notion&logoColor=white" alt="Notion" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/Webhooks-FF9800?style=flat-square" alt="Webhooks" />
</p>

---

## Quick Start

### Option 1: Docker (Recommended)

```bash
git clone https://github.com/humayun-sarfraz/n8n-ai-workflow-kit.git
cd n8n-ai-workflow-kit

cp .env.example .env
# Edit .env with your API keys

docker compose up -d
# Open http://localhost:5678
```

### Option 2: npm

```bash
npm install -g n8n
n8n start
# Open http://localhost:5678
```

### Import a Workflow

1. Open n8n UI at `http://localhost:5678`
2. Go to **Workflows** > **Import from File**
3. Select any JSON file from the `workflows/` folder
4. Assign your credentials to each node
5. Click **Test Workflow** to verify
6. Toggle **Active** to enable

---

## Testing with Webhooks

```bash
# Lead Qualification
curl -X POST http://localhost:5678/webhook/lead-qualification \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/lead-submission.json

# Support Ticket Classifier
curl -X POST http://localhost:5678/webhook/support-ticket \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/support-ticket.json

# Meeting Notes
curl -X POST http://localhost:5678/webhook/meeting-notes \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/meeting-transcript.json

# Invoice Extraction
curl -X POST http://localhost:5678/webhook/invoice-extraction \
  -H "Content-Type: application/json" \
  -d @examples/webhook-payloads/invoice-text.json
```

---

## Project Structure

```
n8n-ai-workflow-kit/
├── workflows/              # 8 ready-to-import n8n workflow JSON templates
├── docs/                   # Setup, credentials, security, troubleshooting
├── prompts/                # Reusable OpenAI prompt templates
├── examples/
│   ├── webhook-payloads/   # Sample JSON payloads for testing
│   └── sample-outputs/     # Example AI-generated outputs
├── .env.example            # Environment variable template
├── docker-compose.yml      # Local n8n setup
└── LICENSE
```

---

## Documentation

| Document | Description |
|:---------|:------------|
| [Setup Guide](docs/setup-guide.md) | Install and configure n8n locally |
| [Credentials Guide](docs/credentials-guide.md) | Configure OpenAI, Gmail, Slack, Notion, CRM credentials |
| [Workflow Import Guide](docs/workflow-import-guide.md) | Step-by-step import and activation |
| [Webhook Examples](docs/webhook-examples.md) | URLs, payloads, and curl commands |
| [Prompt Library](docs/prompt-library.md) | Reusable AI prompts for all workflows |
| [Error Handling Patterns](docs/error-handling-patterns.md) | Retry logic, error alerts, validation |
| [Security Best Practices](docs/security-best-practices.md) | API key safety, PII, data sanitization |
| [Troubleshooting](docs/troubleshooting.md) | Common issues and fixes |

---

## Security

- **Never hardcode API keys** — use n8n's built-in credential system
- **Authenticate webhooks** — enable Basic Auth or Header Auth
- **Sanitize inputs** — validate and clean all incoming data
- **Limit PII sent to OpenAI** — strip unnecessary personal data
- **Human approval** — required for all customer-facing AI responses
- **Rotate credentials** on a regular schedule

---

## Customization

| What | How |
|------|-----|
| **AI Model** | Change `gpt-4o` to `gpt-4o-mini` or any model in OpenAI nodes |
| **Prompts** | Edit system/user messages in nodes or use `prompts/` templates |
| **Integrations** | Swap Slack for Teams, Gmail for Outlook, Sheets for Airtable |
| **Thresholds** | Adjust lead score cutoffs, invoice limits, urgency rules |
| **Nodes** | Add processing, storage, or notification steps to any workflow |

---

## Troubleshooting

| Issue | Solution |
|:------|:---------|
| OpenAI credential error | Verify API key in n8n credentials, check billing status |
| Gmail trigger not firing | Ensure OAuth2 is configured, Gmail API enabled in GCP |
| Webhook not receiving data | Use test mode first, verify URL matches exactly |
| JSON parse error from AI | Add error handling branch, validate output format |
| Rate limit (429) errors | Enable retry logic on OpenAI nodes, add delay between calls |

See [docs/troubleshooting.md](docs/troubleshooting.md) for the full guide.

---

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-workflow`)
3. Commit your changes
4. Push to the branch (`git push origin feature/new-workflow`)
5. Open a Pull Request

---

## License

This project is licensed under the [MIT License](LICENSE).

---

<p align="center">
  <strong>Built for the n8n community</strong><br/>
  <sub>If this project helps you, consider giving it a star!</sub>
</p>
