# Setup Guide

This guide walks you through setting up the n8n AI Workflow Automation Kit from scratch on your local machine or a self-hosted server.

---

## Prerequisites

Before you begin, make sure the following are installed on your system:

| Tool | Minimum Version | Notes |
|------|----------------|-------|
| Node.js | 18.x or higher | Required for npm-based n8n install |
| npm | 9.x or higher | Comes with Node.js |
| Docker | 20.x or higher | Required for Docker-based setup |
| Docker Compose | v2.x | Included with Docker Desktop |
| Git | Any recent version | For cloning this repository |

Verify your installations:

```bash
node --version
npm --version
docker --version
docker compose version
git --version
```

---

## Option A: Local n8n via npm

This approach is fastest for development and local testing.

### 1. Install n8n globally

```bash
npm install -g n8n
```

### 2. Start n8n

```bash
n8n start
```

n8n will be available at `http://localhost:5678`.

### 3. Set environment variables before starting

```bash
export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD=yourpassword
export N8N_HOST=localhost
export N8N_PORT=5678
export N8N_PROTOCOL=http

n8n start
```

### 4. Persist your data directory

By default, n8n stores data in `~/.n8n`. To use a custom directory:

```bash
export N8N_USER_FOLDER=/path/to/your/n8n-data
n8n start
```

---

## Option B: Docker Compose (Recommended)

Docker Compose gives you a reproducible, isolated environment with persistent storage.

### 1. Clone this repository

```bash
git clone https://github.com/humayun-sarfraz/n8n-ai-workflow-kit.git
cd n8n-ai-workflow-kit
```

### 2. Copy the environment file

```bash
cp .env.example .env
```

### 3. Edit `.env` with your values

Open `.env` in your editor and fill in the required values (see the section below).

### 4. Start the stack

```bash
docker compose up -d
```

n8n will be available at `http://localhost:5678`.

### 5. View logs

```bash
docker compose logs -f n8n
```

### 6. Stop the stack

```bash
docker compose down
```

To also remove volumes (wipes all data):

```bash
docker compose down -v
```

---

## Environment Variables Configuration

The following variables are used in `docker-compose.yml` and `.env`. A full `.env.example` is included in the project root.

### Core n8n Settings

```env
# Basic auth for the n8n UI
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=your_secure_password

# Host and port
N8N_HOST=localhost
N8N_PORT=5678
N8N_PROTOCOL=http

# Webhook base URL (must be publicly accessible for webhook triggers)
WEBHOOK_URL=https://your-domain.com/

# Timezone
GENERIC_TIMEZONE=America/New_York

# Encryption key for credentials (generate a random string)
N8N_ENCRYPTION_KEY=your_random_32_char_string_here
```

### Database (SQLite default, PostgreSQL optional)

```env
# SQLite (default, no extra config needed)
DB_TYPE=sqlite

# PostgreSQL (optional, recommended for production)
# DB_TYPE=postgresdb
# DB_POSTGRESDB_HOST=postgres
# DB_POSTGRESDB_PORT=5432
# DB_POSTGRESDB_DATABASE=n8n
# DB_POSTGRESDB_USER=n8n
# DB_POSTGRESDB_PASSWORD=your_db_password
```

### Execution Settings

```env
# Keep execution history (in hours)
EXECUTIONS_DATA_PRUNE=true
EXECUTIONS_DATA_MAX_AGE=168

# Max execution timeout (seconds)
N8N_EXECUTION_TIMEOUT=300
N8N_EXECUTION_TIMEOUT_MAX=600
```

---

## Creating Credentials in n8n

Once n8n is running, you need to add credentials for each service your workflows use.

### General Steps

1. Open n8n at `http://localhost:5678`
2. Click **Settings** (gear icon) in the left sidebar
3. Select **Credentials**
4. Click **Add Credential**
5. Search for the credential type and follow the prompts

### OpenAI

1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Create a new secret key
3. In n8n: Add Credential > **OpenAI** > paste your API key

### Gmail

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project, enable the Gmail API
3. Create OAuth2 credentials (Desktop or Web application)
4. In n8n: Add Credential > **Gmail OAuth2** > enter Client ID and Secret > authorize

### Google Sheets

Same OAuth2 setup as Gmail. Enable the Google Sheets API in Google Cloud Console.

In n8n: Add Credential > **Google Sheets OAuth2** > authorize with your Google account.

### Slack

1. Go to [api.slack.com/apps](https://api.slack.com/apps) and create an app
2. Add Bot Token Scopes: `chat:write`, `channels:read`, `users:read`
3. Install the app to your workspace
4. Copy the **Bot User OAuth Token**
5. In n8n: Add Credential > **Slack** > paste the token

### Notion

1. Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
2. Create a new integration and copy the **Internal Integration Secret**
3. Share the relevant Notion pages/databases with the integration
4. In n8n: Add Credential > **Notion API** > paste the secret

---

## Importing Workflow JSON Files

All workflow templates are in the `workflows/` folder of this repository.

1. Open n8n UI at `http://localhost:5678`
2. Click **Workflows** in the left sidebar
3. Click the **+** button or **New** to open the workflow list options
4. Select **Import from File**
5. Navigate to `workflows/` and select any `.json` file
6. The workflow will open in the editor
7. Assign credentials to each node that requires them (nodes with missing credentials show a warning badge)
8. Save the workflow

For bulk import via CLI:

```bash
# Import all workflows from the workflows/ directory
for f in workflows/*.json; do
  n8n import:workflow --input="$f"
done
```

---

## Testing Workflows

### Using the n8n UI

1. Open the workflow in the editor
2. Click **Test workflow** (play button) in the top right
3. For webhook-triggered workflows, use the **Test URL** shown in the Webhook node
4. Check the **Execution Log** panel at the bottom for output data at each node

### Using curl for Webhook Triggers

```bash
curl -X POST http://localhost:5678/webhook-test/your-webhook-path \
  -H "Content-Type: application/json" \
  -d '{"name": "Test User", "email": "test@example.com"}'
```

### Checking Executions

1. Click **Executions** in the left sidebar
2. Review past runs, their status (success/error), and per-node data
3. Click any execution to inspect input/output at each step

---

## Tips

- Always use the **Test URL** (webhook-test) during development. Switch to the **Production URL** only after activating the workflow.
- Use n8n's built-in **Code** node for custom JavaScript transformations before or after AI nodes.
- Pin test data on nodes using right-click > **Pin Data** to replay executions without re-triggering the source.
- Keep workflows focused. Split complex pipelines into sub-workflows using the **Execute Workflow** node.
