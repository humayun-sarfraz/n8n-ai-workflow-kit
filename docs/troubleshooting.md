# Troubleshooting Guide

This guide covers the most common issues encountered when setting up and operating the n8n AI Workflow Automation Kit, along with step-by-step solutions.

---

## 1. OpenAI Credential Error

### Symptoms

- Node shows: `"Authorization" header is required` or `Incorrect API key provided`
- Execution fails immediately at the OpenAI node
- Error code `401` in the node output

### Causes and Fixes

**Wrong API key:**
1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Verify the key you added is still active (not revoked)
3. In n8n: **Settings > Credentials** > open the OpenAI credential > re-paste the key > Save

**Expired or deleted key:**
- Create a new API key on the OpenAI platform and update the n8n credential

**No credit / exceeded quota:**
- Log in to OpenAI and check **Usage > Billing**
- Add payment method or top up credits
- Error message will typically say `You exceeded your current quota`

**Wrong model name:**
- In the OpenAI node, verify the model field. Valid values: `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, `gpt-3.5-turbo`
- If you have a typo (e.g., `gpt4-o`), you'll get a `model_not_found` error

---

## 2. Gmail Trigger Not Firing

### Symptoms

- The Gmail Trigger node shows "Listening..." but no emails trigger the workflow
- Workflow never executes even after emails arrive in the inbox

### Causes and Fixes

**OAuth credential expired:**
1. In n8n: **Settings > Credentials** > open the Gmail OAuth2 credential
2. Click **Reconnect** or **Connect my account** to refresh the OAuth token
3. Re-authorize with Google

**Wrong label/filter configured:**
- Check the Gmail Trigger node settings: what label or filter is set?
- If set to `INBOX` and emails are being auto-labeled and skipped, adjust the filter
- Test by sending an email that clearly matches the trigger conditions

**Gmail API not enabled:**
1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Select your project
3. Go to **APIs & Services > Enabled APIs**
4. Confirm **Gmail API** is listed; if not, enable it

**Polling interval:**
- The Gmail Trigger polls for new emails on an interval (default: every minute)
- If you just imported the workflow, wait at least 1-2 minutes after activation
- Do not expect immediate triggering — polling triggers are not real-time

**Workflow not activated:**
- The Gmail Trigger only fires in production when the workflow is **Active** (green toggle)
- In test mode, you must click "Listen for Event" manually in the node panel

---

## 3. Google Sheets Permission Denied

### Symptoms

- Error: `The caller does not have permission`
- Error: `Requested entity was not found`
- HTTP 403 or 404 from the Google Sheets node

### Causes and Fixes

**Sheet not shared with OAuth account:**
- The OAuth token authorizes a specific Google account
- Confirm the sheet is owned by or shared with that account
- Open the sheet URL while logged in as that account — if you can't see it, n8n can't either

**Wrong Spreadsheet ID:**
- Open the sheet in your browser
- The URL format is: `https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`
- Copy the exact ID string and paste it into the Google Sheets node

**Wrong Sheet Name (tab name):**
- The Sheet Name field is case-sensitive
- Confirm the tab name in the spreadsheet matches exactly (e.g., `Sheet1` vs `sheet1`)

**Google Sheets API not enabled:**
1. In Google Cloud Console, go to **APIs & Services > Library**
2. Search **Google Sheets API** and enable it for your project
3. This is separate from enabling the Google Drive API

**Token scope issue:**
- Delete the Google Sheets OAuth2 credential in n8n and recreate it
- During the OAuth flow, make sure you approve all requested scopes

---

## 4. Slack Message Not Sending

### Symptoms

- Error: `channel_not_found`
- Error: `not_in_channel`
- Error: `missing_scope`
- Message appears to send (green node) but nothing arrives in Slack

### Causes and Fixes

**Bot not invited to the channel:**
- In Slack, open the target channel
- Type `/invite @your-bot-name` and press Enter
- Try sending again

**Wrong channel name format:**
- Use the channel name without `#`: `general` not `#general`
- Or use the channel ID (right-click channel > View channel details > Copy channel ID)
- Channel IDs are more reliable: they start with `C` for public channels

**Missing bot scopes:**
1. Go to [api.slack.com/apps](https://api.slack.com/apps) > your app > **OAuth & Permissions**
2. Verify `chat:write` is listed under Bot Token Scopes
3. If missing: add it, then reinstall the app to your workspace
4. Copy the new Bot User OAuth Token and update the n8n credential

**Wrong token type:**
- Use the **Bot User OAuth Token** (starts with `xoxb-`)
- Do not use the User OAuth Token (starts with `xoxp-`) or the App-Level Token (`xapp-`)

---

## 5. Webhook Test Mode vs Production Mode Confusion

### Symptoms

- curl requests return 404 during development
- Workflow works in editor but stops after closing the browser tab
- External system gets no response after workflow activation

### Explanation and Fix

n8n has two webhook URL types that behave completely differently:

| | Test URL | Production URL |
|--|----------|---------------|
| Path | `/webhook-test/{path}` | `/webhook/{path}` |
| Active when | Editor is open + "Listen" clicked | Workflow is **Activated** |
| Used for | Development and testing only | Real integrations |

**Fix for "curl returns 404 in development":**
- Open the workflow editor
- Click the Webhook node
- Click **Listen for Test Event**
- While it's listening, send your curl request to the **test URL**

**Fix for "stops working after closing editor":**
- Activate the workflow (toggle switch to Active)
- Update your external system to use the **production URL** (`/webhook/`)

**Fix for "404 on production URL":**
- Confirm the workflow is **Active** (not just saved)
- Confirm you are using `/webhook/` not `/webhook-test/`
- Confirm the path in the URL matches the **Path** field in the Webhook node exactly

---

## 6. JSON Parsing Errors from AI Output

### Symptoms

- Code node throws `SyntaxError: Unexpected token`
- Error: `Cannot read properties of undefined (reading 'choices')`
- AI node succeeds but downstream parsing fails

### Causes and Fixes

**AI returned text instead of JSON:**
- The OpenAI node's response is in `choices[0].message.content` as a string
- If you asked for JSON but the model included explanatory text or markdown fences, parsing fails

**Fix 1 — Enable JSON mode:**
- In the OpenAI node, find the **Response Format** option
- Set it to **JSON Object** (`response_format: { type: "json_object" }`)
- This forces OpenAI to return only valid JSON

**Fix 2 — Use the safe JSON parse pattern from `error-handling-patterns.md`:**

```javascript
const content = $input.first().json.choices[0].message.content;
let parsed;
try {
  parsed = JSON.parse(content);
} catch (e) {
  // Strip markdown fences and try again
  const clean = content.replace(/```json\n?|\n?```/g, '').trim();
  parsed = JSON.parse(clean);
}
return [{ json: parsed }];
```

**Fix 3 — Update the system prompt:**
- Add to the system prompt: `"Respond ONLY with a valid JSON object. Do not include any explanation, markdown formatting, or code fences."`

---

## 7. Rate Limit Errors (429)

### Symptoms

- Error: `429 Too Many Requests`
- Error: `You exceeded your current quota`
- Error: `Rate limit reached for model`

### Causes and Fixes

**OpenAI rate limits:**
- Every OpenAI model has rate limits (requests per minute, tokens per minute)
- Tier 1 accounts have low limits; higher tiers have more capacity

**Fix 1 — Enable retry with wait:**
- On the OpenAI node: enable **Retry on Fail**, set **Max Tries** to 5, **Wait** to 10000ms

**Fix 2 — Upgrade OpenAI tier:**
- Pay OpenAI more (pre-purchase credits) to get your account moved to a higher rate limit tier
- Check your current tier at [platform.openai.com/account/limits](https://platform.openai.com/account/limits)

**Fix 3 — Reduce concurrent executions:**
- In n8n `.env`, set `N8N_CONCURRENCY_PRODUCTION_LIMIT=2` to limit parallel workflow executions
- This prevents bursts of simultaneous OpenAI calls

**Fix 4 — Add a Wait node:**
- For batch processing workflows, add a **Wait** node between iterations
- Set it to wait 2-5 seconds between each AI call

**Other APIs (Slack, HubSpot, Google):**
- Each has its own rate limits; check their documentation
- Same retry pattern applies: enable Retry on Fail with a 2-5 second wait

---

## 8. n8n Docker Container Not Starting

### Symptoms

- `docker compose up -d` completes but container exits immediately
- `docker compose ps` shows container status as `Exit 1` or `Exit 137`
- Browser shows connection refused at `localhost:5678`

### Diagnosis

```bash
# View the container logs
docker compose logs n8n

# Check container status
docker compose ps
```

### Common Causes and Fixes

**Port already in use:**
```
Error: listen EADDRINUSE: address already in use :::5678
```
- Another process is using port 5678
- Find and stop it: `netstat -ano | findstr :5678` (Windows) or `lsof -i :5678` (Mac/Linux)
- Or change the port in `docker-compose.yml`: `"5679:5678"` and access via port 5679

**Volume permission error:**
```
Error: EACCES: permission denied, mkdir '/home/node/.n8n'
```
- Run: `sudo chown -R 1000:1000 ./n8n-data` (replace `./n8n-data` with your volume path)

**Invalid environment variable:**
- Check your `.env` file for typos or missing values
- Run `docker compose config` to see the resolved configuration before starting

**Missing `.env` file:**
- Copy `.env.example` to `.env` and fill in all required values
- Docker Compose will fail silently or use empty values if `.env` is missing

**Encryption key changed:**
- If you change `N8N_ENCRYPTION_KEY` after credentials have been saved, n8n cannot decrypt them
- Never change the encryption key on an existing data directory without first exporting and re-importing all credentials

---

## 9. Workflow Not Activating

### Symptoms

- Toggle switch turns green but then reverts to inactive
- Error appears when trying to activate: `Your workflow has errors`
- Workflow saves but does not respond to triggers after activation

### Causes and Fixes

**Workflow has validation errors:**
- n8n shows a red badge on nodes with issues
- Look for orange warning badges (missing credentials) or red badges (invalid configuration)
- Fix all flagged nodes before activating

**Trigger node not configured:**
- Every active workflow must start with a trigger node (Webhook, Schedule, Gmail Trigger, etc.)
- If the workflow starts with a manual trigger, it cannot be activated — only executed manually

**Duplicate webhook path:**
- Two active workflows cannot share the same webhook path
- Check your other active workflows for path conflicts
- Error message: `There is already a workflow with this webhook`

**License or plan limitation:**
- n8n Community Edition allows unlimited workflows but has some execution limits
- n8n Cloud plans may have active workflow limits on lower tiers

---

## 10. Timeout Errors on Long-Running Workflows

### Symptoms

- Error: `Workflow execution timed out`
- Error: `The execution exceeded the maximum allowed time`
- Long-running nodes (OpenAI with large prompts, bulk Sheet writes) fail after a set time

### Causes and Fixes

**Default timeout too short:**
- n8n has a default execution timeout (usually 300 seconds / 5 minutes)

**Fix — Increase timeout via environment variables:**

```env
# In .env or docker-compose.yml environment section
N8N_EXECUTION_TIMEOUT=600
N8N_EXECUTION_TIMEOUT_MAX=3600
```

**Fix — Break workflows into smaller pieces:**
- Use the **Execute Workflow** node to chain sub-workflows
- Each sub-workflow has its own timeout window
- This also makes debugging easier since you can see each step separately

**Fix — Use async patterns for large batches:**
- Instead of processing 1000 rows in one execution, trigger individual executions per row using the **Split In Batches** node
- Process 10-50 items per batch with a Wait node between batches

**OpenAI slow response:**
- Large prompts with long context windows take longer to process
- Consider splitting large documents and summarizing sections independently before a final summary pass
- Use `gpt-4o-mini` for faster (cheaper) responses where full GPT-4o quality is not required

---

## General Debugging Tips

- **Execution log:** Always start debugging by clicking **Executions** in the left sidebar. Open the failed execution and click each node to see exact input/output data.
- **Pin data:** Right-click any node > **Pin Data** to replay the workflow with the same test data without re-triggering the source.
- **Isolate nodes:** Disable upstream nodes temporarily (right-click > **Disable**) to test a specific part of the workflow in isolation.
- **n8n community forum:** [community.n8n.io](https://community.n8n.io) — search for your exact error message before opening a new thread.
- **Check n8n version:** Run `n8n --version` or check the Docker image tag. Some bugs are version-specific and resolved by updating.
- **Browser console:** For UI-level issues, open your browser's developer tools (F12) and check the console for additional error details that may not appear in the n8n UI.
