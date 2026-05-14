# Credentials Guide

This guide explains how to configure every credential type used in the n8n AI Workflow Automation Kit. All credentials are stored encrypted inside n8n and are never exposed in workflow JSON exports.

---

## General: Adding a Credential in n8n

1. Open n8n at `http://localhost:5678`
2. Click **Settings** (gear icon) in the left sidebar
3. Click **Credentials**
4. Click **Add Credential** (top right)
5. Search for the credential type by name
6. Fill in the required fields and click **Save**

Once saved, the credential appears in the dropdown when you configure any node that uses that service.

---

## OpenAI API Key

**Used by:** AI nodes throughout all workflows (GPT-4, embeddings, etc.)

### Steps

1. Log in to [platform.openai.com](https://platform.openai.com)
2. Navigate to **API Keys** in the left menu
3. Click **Create new secret key**
4. Give it a name (e.g., `n8n-workflow-kit`) and click **Create secret key**
5. Copy the key immediately — it is only shown once

### In n8n

1. Add Credential > search **OpenAI**
2. Select **OpenAI**
3. Paste your key into the **API Key** field
4. Click **Save**

**Tip:** Create a separate API key per project so you can revoke access independently without disrupting other tools.

---

## Gmail OAuth2

**Used by:** Gmail Trigger node, Gmail Send node

### Prerequisites

- A Google account
- A Google Cloud project with the Gmail API enabled

### Steps

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (or select an existing one)
3. Navigate to **APIs & Services > Library**
4. Search for **Gmail API** and click **Enable**
5. Go to **APIs & Services > Credentials**
6. Click **Create Credentials > OAuth client ID**
7. Set Application type to **Web application**
8. Under **Authorized redirect URIs**, add:
   ```
   http://localhost:5678/rest/oauth2-credential/callback
   ```
   Replace `localhost:5678` with your actual n8n URL if self-hosted.
9. Click **Create** and copy the **Client ID** and **Client Secret**

### In n8n

1. Add Credential > search **Gmail**
2. Select **Gmail OAuth2 API**
3. Paste your **Client ID** and **Client Secret**
4. Click **Connect my account** — a Google authorization window opens
5. Sign in and grant permission
6. Click **Save**

**Tip:** Enable the Gmail API under **APIs & Services > Library** — not just the OAuth consent screen. Missing this causes a 403 error.

---

## Google Sheets OAuth2

**Used by:** Google Sheets Read/Write nodes

### Steps

Same OAuth2 application setup as Gmail. In the same Google Cloud project:

1. Go to **APIs & Services > Library**
2. Search for **Google Sheets API** and **Enable** it
3. Reuse the same OAuth client ID and secret from the Gmail setup above, or create a separate one

### In n8n

1. Add Credential > search **Google Sheets**
2. Select **Google Sheets OAuth2 API**
3. Paste **Client ID** and **Client Secret**
4. Click **Connect my account** and authorize
5. Click **Save**

**Tip:** You can use a single OAuth2 Client with multiple Google APIs enabled. n8n will request only the scopes needed by each credential type.

---

## Slack Bot Token / OAuth2

**Used by:** Slack Send Message node, Slack Trigger node

### Steps

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App > From scratch**
3. Name your app and select your workspace
4. Go to **OAuth & Permissions** in the left sidebar
5. Under **Scopes > Bot Token Scopes**, add:
   - `chat:write` — send messages
   - `channels:read` — list public channels
   - `users:read` — look up user info
   - `files:write` — optional, for file uploads
   - `reactions:write` — optional, for emoji reactions
6. Scroll up and click **Install to Workspace**
7. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### In n8n

1. Add Credential > search **Slack**
2. Select **Slack API**
3. Paste the **Bot User OAuth Token** into the **Access Token** field
4. Click **Save**

**Tip:** Invite the bot to channels before trying to send messages. In Slack, type `/invite @your-bot-name` in the target channel.

---

## Notion API Key

**Used by:** Notion nodes for reading/writing pages and databases

### Steps

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Click **New integration**
3. Name it (e.g., `n8n-integration`) and select your workspace
4. Under **Capabilities**, enable:
   - Read content
   - Update content
   - Insert content
5. Click **Submit**
6. Copy the **Internal Integration Secret** (starts with `secret_`)

### Share Pages with the Integration

For each Notion page or database your workflow accesses:

1. Open the page in Notion
2. Click the **...** menu (top right)
3. Go to **Connections** (or **Add connections** in older UI)
4. Search for your integration name and click to share

### In n8n

1. Add Credential > search **Notion**
2. Select **Notion API**
3. Paste the **Internal Integration Secret**
4. Click **Save**

**Tip:** Notion integrations do not have access to any pages by default. You must explicitly share each page or database with the integration — otherwise you get a 404 or "object not found" error.

---

## CRM Credentials

### HubSpot

1. Log in to HubSpot
2. Go to **Settings > Integrations > Private Apps**
3. Click **Create a private app**
4. Under **Scopes**, add:
   - `crm.objects.contacts.read`
   - `crm.objects.contacts.write`
   - `crm.objects.deals.read`
   - `crm.objects.deals.write`
5. Click **Create app** and copy the **Access Token**

**In n8n:**
1. Add Credential > **HubSpot API**
2. Paste the Access Token
3. Save

### Pipedrive

1. Log in to Pipedrive
2. Go to **Settings > Personal preferences > API**
3. Copy your **Personal API Token**

**In n8n:**
1. Add Credential > **Pipedrive API**
2. Paste the API Token
3. Save

### Salesforce

Salesforce uses OAuth2 with a Connected App.

1. In Salesforce, go to **Setup > App Manager > New Connected App**
2. Enable **OAuth Settings**
3. Set the callback URL to your n8n OAuth callback URL
4. Add scopes: `Full access` or specific API scopes
5. Save and note the **Consumer Key** (Client ID) and **Consumer Secret**

**In n8n:**
1. Add Credential > **Salesforce OAuth2 API**
2. Enter **Client ID**, **Client Secret**, and your Salesforce instance URL
3. Click **Connect** and authorize
4. Save

---

## Webhook Authentication

Webhooks exposed by n8n can optionally require authentication to prevent unauthorized calls.

### Basic Authentication

In the Webhook node settings:

1. Set **Authentication** to **Basic Auth**
2. Set a **Username** and **Password**

Callers must include an `Authorization` header:

```bash
curl -X POST https://your-n8n.com/webhook/your-path \
  -H "Authorization: Basic $(echo -n 'username:password' | base64)" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### Header Authentication

1. Set **Authentication** to **Header Auth**
2. Set a **Header Name** (e.g., `X-Webhook-Secret`) and a **Header Value** (your secret token)

Callers must include the header:

```bash
curl -X POST https://your-n8n.com/webhook/your-path \
  -H "X-Webhook-Secret: your_secret_token" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

**Tip:** Store webhook secrets in n8n environment variables and reference them in your webhook node configuration. Do not hardcode secrets in workflow JSON.

---

## Credential Security Checklist

- [ ] API keys are added as n8n credentials, not pasted into node parameters
- [ ] OAuth tokens have only the minimum required scopes
- [ ] Webhook nodes use Basic Auth or Header Auth in production
- [ ] Notion integrations are shared only with specific pages, not entire workspaces
- [ ] Slack bot scopes are limited to what workflows actually need
- [ ] HubSpot private app scopes match the operations in your workflows
- [ ] Salesforce Connected App scopes are as narrow as possible
- [ ] All credentials are rotated after any suspected exposure
