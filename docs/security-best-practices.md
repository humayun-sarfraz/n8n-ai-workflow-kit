# Security Best Practices

This document outlines security practices for deploying and operating the n8n AI Workflow Automation Kit. Following these guidelines protects your data, your integrations, and your users.

---

## 1. Never Hardcode API Keys — Use n8n Credentials

**Do not** paste API keys, tokens, or passwords directly into node parameters, Code nodes, or workflow JSON files.

**Why it matters:** Workflow JSON files can be exported, shared, or accidentally committed to version control — exposing secrets to anyone who can read the file.

**What to do instead:**

- Add all secrets through **Settings > Credentials** in the n8n UI
- Select the credential from the dropdown within each node
- For secrets needed in Code nodes, use n8n environment variables referenced via `process.env.MY_SECRET` — set these in your `.env` file or Docker Compose environment block, never in code

**Example — correct way to use a secret in a Code node:**

```javascript
// CORRECT: Read from environment variable
const apiKey = process.env.MY_API_KEY;

// WRONG: Never do this
const apiKey = "sk-abc123hardcoded";
```

**Audit:** Periodically export your workflows and search the JSON for patterns like `sk-`, `Bearer `, `password`, or `secret`. If found, rotate those keys immediately and move them to credentials.

---

## 2. Use Webhook Authentication (Basic Auth, Header Auth)

Any n8n webhook URL is publicly reachable by anyone who knows the URL. Always require authentication on production webhooks.

### Recommended: Header Authentication

In each Webhook node:

1. Set **Authentication** to **Header Auth**
2. Set **Header Name**: `X-Webhook-Secret`
3. Set **Header Value**: a randomly generated secret (minimum 32 characters)

Generate a strong secret:

```bash
openssl rand -hex 32
```

Store the secret in your environment file and configure your sending systems to include it in every request.

### When to Use Basic Auth

Use Basic Auth when integrating with systems that support it natively but not custom headers (e.g., some older form tools or payment processors).

### Disable Unauthenticated Webhooks in Production

Review all active webhook workflows and confirm none have **Authentication** set to **None** unless the route is already protected at the reverse proxy level (e.g., Nginx, Cloudflare Access).

---

## 3. Sanitize Incoming Webhook Data

Never trust data received from external sources. Validate and sanitize all webhook payloads before passing them to AI nodes, databases, or messaging services.

### What to Check

- **Field presence:** Verify required fields exist before using them
- **Field types:** Confirm strings are strings, numbers are numbers
- **Length limits:** Reject payloads where a single field is excessively long (could indicate injection attempts)
- **Allowed characters:** For fields used in commands or queries, strip or escape special characters

### Sanitization Code Node Example

```javascript
const raw = $input.first().json;

// Strip HTML tags from text fields
const stripHtml = (str) => typeof str === 'string'
  ? str.replace(/<[^>]*>/g, '').trim()
  : '';

// Truncate strings to reasonable lengths
const truncate = (str, max) => typeof str === 'string'
  ? str.substring(0, max)
  : '';

const sanitized = {
  name: stripHtml(truncate(raw.name, 100)),
  email: truncate(raw.email, 254).toLowerCase(),
  company: stripHtml(truncate(raw.company, 200)),
  message: stripHtml(truncate(raw.message, 5000)),
};

// Reject if required fields are missing after sanitization
if (!sanitized.email || !sanitized.name) {
  throw new Error('Required fields missing or invalid after sanitization');
}

return [{ json: sanitized }];
```

---

## 4. Limit Sensitive Data Sent to OpenAI (PII Considerations)

When data passes through OpenAI's API, it leaves your infrastructure. By default, OpenAI does not train on API data (per their API terms), but you should still minimize PII exposure.

### PII Reduction Guidelines

- **Do not send full names + email + company together** unless the AI specifically needs all three to perform its task
- **Anonymize where possible:** Replace actual names with roles (`Customer`, `Requester`) before sending to OpenAI
- **Do not send financial account numbers, SSNs, passwords, or health data** to any third-party AI API
- **Truncate long text:** If the AI only needs to classify a message, send the first 500 characters rather than a full 5000-character body

### Anonymization Code Node Example

```javascript
const data = $input.first().json;

// For classification tasks, strip identifying info before sending to AI
const aiPayload = {
  // Keep: the text content needed for classification
  subject: data.subject,
  body: data.body.substring(0, 1000),
  plan_tier: data.customer_plan,  // non-identifying business context
  
  // Omit: PII not needed for classification
  // customer_name, customer_email, customer_id — retained separately
};

return [{
  json: {
    aiPayload: aiPayload,
    // Preserve PII locally for post-AI use
    customerRef: {
      name: data.customer_name,
      email: data.customer_email
    }
  }
}];
```

---

## 5. Use Human Approval for External-Facing Responses

Never let an AI generate a response that goes directly to a customer or partner without human review.

### Human-in-the-Loop Pattern

```
Webhook → AI Draft Response → Gmail: Send to Internal Reviewer
                                        ↓
                              Wait for Approval (Webhook or Form)
                                        ↓
                              IF Approved → Send to Customer
                              IF Rejected → Discard or Escalate
```

### Implementation Options

- **n8n Wait node:** Pause the workflow execution until a callback URL is called (approve/reject)
- **Approval email with links:** Send the draft to an internal reviewer with "Approve" and "Reject" links that call back to different n8n webhook paths
- **Slack interactive buttons:** Use Slack Block Kit buttons in the alert message so reviewers can approve/reject directly in Slack

### When Human Approval Can Be Skipped

- Internal notifications (alerts, logs) that do not reach external parties
- Data transformation tasks with no external output
- Read-only operations (fetching data, classification without output)

---

## 6. Log Only Necessary Information

Execution logs in n8n contain full node input/output data. This data persists in the n8n database.

### Reduce Sensitive Data in Execution Logs

- **Set execution data pruning:** In your `.env`, set `EXECUTIONS_DATA_PRUNE=true` and `EXECUTIONS_DATA_MAX_AGE=72` (keep logs for 72 hours, not indefinitely)
- **Mask sensitive fields** in Code nodes before passing to downstream nodes that might log them
- **Avoid logging full payloads** in Slack alert messages — log only identifiers (ticket ID, workflow name) and link to the n8n execution for full details

### Execution Pruning Configuration

```env
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=72
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS=true
```

Setting `EXECUTIONS_DATA_SAVE_ON_SUCCESS=none` means successful executions are not stored — only failures. This dramatically reduces stored PII.

---

## 7. Apply Least Privilege to Integrations

Each integration should have only the permissions it needs for its specific workflows.

| Service | What to Restrict |
|---------|----------------|
| OpenAI | Use a project-scoped API key, not your personal account key |
| Gmail OAuth | Request only `gmail.send` and `gmail.readonly` scopes, not full account access |
| Google Sheets | Share only the specific sheets used by workflows, not entire drives |
| Slack | Limit bot token scopes to `chat:write`, `channels:read` — not `admin` or `users:write` |
| Notion | Share only the specific databases used by workflows |
| HubSpot | Create a private app with only the contact/deal CRM scopes needed |
| n8n itself | Create separate user accounts for team members; avoid sharing the admin account |

### Creating Separate n8n Users

In n8n (version 0.227+), go to **Settings > Users** to add team members with appropriate roles (Owner, Member). This ensures audit trails track who changed which workflow.

---

## 8. Docker Security

If running n8n via Docker, apply these hardening measures.

### Run as Non-Root User

In your `docker-compose.yml`, specify a non-root user:

```yaml
services:
  n8n:
    image: n8nio/n8n
    user: "1000:1000"
    # ...
```

Alternatively, the official n8n Docker image already runs as the `node` user (UID 1000) by default.

### Network Isolation

Create a dedicated Docker network so n8n only communicates with services it needs:

```yaml
services:
  n8n:
    networks:
      - n8n-internal

  postgres:
    networks:
      - n8n-internal

networks:
  n8n-internal:
    driver: bridge
    internal: false  # set to true if n8n should not reach the internet (unusual)
```

### Do Not Expose Docker Socket

Never mount the Docker socket (`/var/run/docker.sock`) into the n8n container unless you have a specific reason. It grants container escape capabilities.

### Use Secrets for Sensitive Environment Variables

Instead of plain environment variables in compose files, use Docker secrets for production:

```yaml
services:
  n8n:
    environment:
      - N8N_ENCRYPTION_KEY_FILE=/run/secrets/n8n_encryption_key
    secrets:
      - n8n_encryption_key

secrets:
  n8n_encryption_key:
    file: ./secrets/n8n_encryption_key.txt
```

### Keep the n8n Image Updated

```bash
docker compose pull
docker compose up -d
```

Pin to a specific version tag in production rather than `latest` to control when updates happen:

```yaml
image: n8nio/n8n:1.45.0
```

---

## 9. Rotate Credentials Regularly

Credentials that are never rotated become a permanent risk if they are silently compromised.

### Rotation Schedule

| Credential Type | Recommended Rotation |
|----------------|---------------------|
| OpenAI API keys | Every 90 days or on any suspected exposure |
| Webhook secrets | Every 90 days |
| n8n encryption key | On any suspected infrastructure compromise only (requires re-entering all credentials) |
| OAuth tokens | Managed by the provider; revoke and reauthorize if access is revoked |
| HubSpot / Pipedrive tokens | Every 180 days |
| Slack bot tokens | On team member offboarding or app permission changes |

### How to Rotate Without Downtime

1. Create the new credential value in the external service
2. Add a **new** n8n credential entry with the updated value
3. Update all affected nodes to use the new credential (node by node)
4. Test each updated workflow
5. Delete the old credential entry
6. Revoke the old key in the external service

---

## Security Checklist

- [ ] All API keys stored in n8n credentials, not hardcoded in workflows
- [ ] All production webhook nodes have authentication enabled
- [ ] Incoming webhook data is validated and sanitized before processing
- [ ] PII is minimized before sending data to OpenAI or other third-party AI APIs
- [ ] Human approval gate exists for all customer-facing AI-generated responses
- [ ] Execution log pruning is configured to limit stored sensitive data
- [ ] All integrations use least-privilege scopes
- [ ] n8n Docker container runs as non-root user
- [ ] Docker network isolation is configured
- [ ] n8n image version is pinned and regularly updated
- [ ] Credential rotation schedule is documented and followed
- [ ] Team members have individual n8n accounts (no shared admin credentials)
