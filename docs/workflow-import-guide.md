# Workflow Import Guide

This guide walks you through importing workflow JSON templates from this kit into your n8n instance, assigning credentials, and activating the workflows.

---

## Overview

Each workflow in the `workflows/` directory is a self-contained JSON file. Importing it gives you all the nodes, connections, and settings — but you must still assign your own credentials to authenticated nodes.

---

## Step 1: Open n8n UI

Open your browser and navigate to your n8n instance:

- Local: `http://localhost:5678`
- Docker: `http://localhost:5678` (or whatever port you mapped)
- Self-hosted: `https://your-domain.com`

Log in with your admin credentials.

---

## Step 2: Go to Workflows

In the left sidebar, click **Workflows**.

You will see a list of all existing workflows (empty if this is a fresh install).


---

## Step 3: Open the Import Dialog

Click the **+** button (top right) or the **New** button to reveal workflow creation options.

Select **Import from File** from the dropdown or menu.


Alternatively, from inside an open workflow editor, click the **...** menu (top right of the editor canvas) and select **Import from File**.

---

## Step 4: Select a JSON Template

A file picker dialog opens.

Navigate to the `workflows/` directory of this repository on your local machine.

Select the JSON file for the workflow you want to import, for example:

```
workflows/01-ai-email-summarizer.json
workflows/02-ai-lead-qualification-and-follow-up.json
workflows/03-support-ticket-classifier.json
workflows/04-meeting-notes-generator.json
workflows/05-google-sheets-crm-enrichment.json
workflows/06-invoice-data-extraction.json
workflows/07-daily-business-report-generator.json
workflows/08-human-approval-ai-response.json
```

Click **Open** (or **Import**).

The workflow opens immediately in the canvas editor.


---

## Step 5: Configure Credentials for Each Node

Nodes that require credentials display an orange warning badge.

For each flagged node:

1. Click the node to open its settings panel
2. In the **Credential** dropdown, select an existing credential or click **Create New Credential**
3. If creating new: follow the credential setup (see `credentials-guide.md`)
4. Once assigned, the warning badge disappears
5. Click outside the panel or press **Escape** to close it

### Common credential assignments by node type

| Node | Credential Type |
|------|----------------|
| OpenAI | OpenAI API Key |
| Gmail Trigger | Gmail OAuth2 API |
| Gmail Send | Gmail OAuth2 API |
| Google Sheets | Google Sheets OAuth2 API |
| Slack | Slack API |
| Notion | Notion API |
| HubSpot | HubSpot API |
| HTTP Request (custom) | None, Basic Auth, or Header Auth depending on the API |

> **Tip:** If you have already added credentials under **Settings > Credentials**, they will appear in the dropdown automatically.

---

## Step 6: Configure Workflow-Specific Settings

Some workflows require you to set node parameters beyond credentials.

### Common parameters to review after import

- **Webhook node** — confirm the path matches what your external system will call
- **Google Sheets node** — set the correct **Spreadsheet ID** and **Sheet Name**
- **Notion node** — set the correct **Database ID**
- **Slack node** — set the target **Channel** name or ID
- **OpenAI node** — review the model (e.g., `gpt-4o`) and adjust the prompt if needed
- **IF / Switch nodes** — verify field names match your actual data structure

To find a Google Sheet ID: open the sheet in your browser. The URL looks like:
```
https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
```

To find a Notion Database ID: open the database, click **Share > Copy link**. The ID is the 32-character string in the URL before the `?`.

---

## Step 7: Test the Trigger

Before activating, run a test to make sure the workflow executes correctly.

### For Webhook-triggered workflows

1. Click the **Webhook** node
2. Copy the **Test URL** shown in the node panel
3. Send a test request using curl or a tool like Postman:

```bash
curl -X POST http://localhost:5678/webhook-test/your-path \
  -H "Content-Type: application/json" \
  -d '{"name": "Test User", "email": "test@example.com", "company": "Acme Corp"}'
```

4. Switch back to n8n — the workflow starts executing in test mode
5. Click **Test workflow** (play button, top right) while the test URL is listening

### For Schedule-triggered workflows

1. Click the **Schedule Trigger** node
2. Temporarily set the interval to every 1 minute for testing
3. Click **Test workflow** to run immediately without waiting
4. Restore the original schedule after verification

### For Email-triggered workflows (Gmail Trigger)

1. Click the **Gmail Trigger** node
2. Click **Listen for event**
3. Send a real email to the monitored inbox
4. The workflow fires once an email arrives

---

## Step 8: Review Execution Results

After running a test:

1. The **Execution Log** panel appears at the bottom of the canvas
2. Click each node to see its input and output data
3. Green nodes = success, Red nodes = error
4. Click a red node to read the error message


Fix any errors — usually:
- Wrong credential assigned
- Incorrect field name in an expression
- Missing required parameter in a node

---

## Step 9: Activate the Workflow

Once testing passes, activate the workflow for production:

1. Toggle the **Inactive / Active** switch in the top right of the editor (it turns green when active)
2. For webhook workflows: the **Production URL** is now live (different from the test URL)
3. For scheduled workflows: the cron timer starts immediately

> **Important:** Webhook **test URLs** (`/webhook-test/`) only work while the workflow editor is open and listening. The **production URL** (`/webhook/`) works only after the workflow is activated. Always switch to production URLs in your external systems after activation.

---

## Tips and Best Practices

- **Import one workflow at a time** and test it fully before importing the next
- **Pin test data** on any node by right-clicking it and selecting **Pin Data** — this lets you replay the workflow without re-triggering the source
- **Duplicate workflows** before making modifications so you keep a clean copy of the original template
- **Use workflow tags** (available in the workflow settings menu) to organize workflows by category: `lead`, `support`, `billing`, `reporting`
- **Sub-workflows:** If a workflow uses the **Execute Workflow** node to call another workflow, import and activate the child workflow first
- **Execution history:** Go to **Executions** in the left sidebar to review all past runs across all workflows

---

## Bulk Import via CLI

To import all workflows at once using the n8n CLI:

```bash
# If n8n is installed globally via npm
for f in workflows/*.json; do
  n8n import:workflow --input="$f"
done

# If running via Docker
for f in workflows/*.json; do
  docker exec -i n8n n8n import:workflow --input="/data/workflows/$(basename $f)"
done
```

Note: You still need to assign credentials after bulk import. The CLI import does not automatically map credentials.
