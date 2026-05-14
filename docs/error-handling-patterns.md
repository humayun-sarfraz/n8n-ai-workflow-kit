# Error Handling Patterns

This document covers how to build robust, fault-tolerant workflows in n8n. It includes configuration examples for retry logic, error trigger workflows, AI-specific failure handling, and input validation.

---

## n8n Error Handling Basics

Every n8n node can fail. When a node fails, n8n by default stops the workflow and marks the execution as failed. You have two tools to change this behavior:

### 1. Node-Level Error Handling

Right-click any node and select **Settings**. Two relevant options:

- **Continue on Fail** — if the node errors, pass the error data downstream instead of stopping the workflow. The next node receives `{ "error": { "message": "...", "description": "..." } }`.
- **Retry on Fail** — automatically retry the node before marking it as failed.

### 2. Workflow-Level Error Trigger

Create a separate "error handling" workflow that gets triggered automatically whenever any other workflow fails.

---

## Retry Logic Configuration

Use retry settings on nodes that call external APIs (OpenAI, Slack, Google Sheets, etc.) since transient network failures are common.

### In the n8n UI

1. Open the node settings panel
2. Click the **Settings** tab
3. Enable **Retry On Fail**
4. Set:
   - **Max Tries**: 3 (recommended for most APIs)
   - **Wait Between Tries (ms)**: 2000 (2 seconds, use 5000+ for rate-limited APIs)

### Equivalent node JSON configuration

```json
{
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 2000
}
```

### Recommended retry settings by service

| Service | Max Tries | Wait (ms) | Notes |
|---------|-----------|-----------|-------|
| OpenAI | 3 | 5000 | Rate limits are common; exponential backoff preferred |
| Gmail | 2 | 3000 | OAuth token issues are usually permanent, retry is shallow |
| Google Sheets | 3 | 2000 | Quota errors benefit from retry |
| Slack | 2 | 1000 | Usually fast, failures are often auth issues |
| HubSpot | 3 | 2000 | Standard API retry |
| Custom HTTP | 3 | 3000 | Adjust based on target API behavior |

---

## Error Trigger Workflow Pattern

This is the most important error handling pattern in n8n. It creates a centralized error handler that fires whenever any activated workflow fails.

### Step 1: Create the Error Handler Workflow

Create a new workflow named `_Error Handler` (the underscore keeps it at the top of the list).

### Step 2: Add the Error Trigger Node

1. Add an **Error Trigger** node (search "Error Trigger" in the node picker)
2. This node receives the failed execution's data automatically:

```json
{
  "execution": {
    "id": "123",
    "url": "http://localhost:5678/execution/123",
    "retryOf": null,
    "error": {
      "message": "The node 'OpenAI' failed: 429 Too Many Requests",
      "stack": "..."
    },
    "lastNodeExecuted": "OpenAI",
    "mode": "production"
  },
  "workflow": {
    "id": "456",
    "name": "Lead Capture AI Qualifier",
    "active": true
  }
}
```

### Step 3: Add a Slack Notification Node

After the Error Trigger, add a **Slack** node configured to send to your `#alerts` channel:

**Message template (Slack markdown):**

```
:rotating_light: *Workflow Error Detected*

*Workflow:* {{ $json.workflow.name }}
*Failed Node:* {{ $json.execution.lastNodeExecuted }}
*Error:* {{ $json.execution.error.message }}
*Execution URL:* {{ $json.execution.url }}
*Time:* {{ $now.toFormat("yyyy-MM-dd HH:mm:ss") }}
```

### Step 4: Optionally Log to Google Sheets

Add a **Google Sheets** node after the Slack node to append the error to an error log sheet:

| Column | Expression |
|--------|-----------|
| Timestamp | `{{ $now.toISO() }}` |
| Workflow | `{{ $json.workflow.name }}` |
| Failed Node | `{{ $json.execution.lastNodeExecuted }}` |
| Error Message | `{{ $json.execution.error.message }}` |
| Execution URL | `{{ $json.execution.url }}` |

### Step 5: Set This as the Error Workflow

For each main workflow:

1. Open the workflow
2. Click the **...** menu > **Settings**
3. Under **Error Workflow**, select `_Error Handler`
4. Save

---

## Slack Error Alert Pattern

For inline (non-Error Trigger) alert patterns — for example, when you want to alert on a specific condition rather than a crash — use this pattern:

```
Webhook → Process Data → IF (check for failure condition)
                              ↓ [true branch]
                         Slack Alert Node
                              ↓
                         Stop and Error Node
                              ↓ [false branch]
                         Continue workflow
```

**IF node condition example** — check if OpenAI returned an empty response:

```
{{ $json.choices[0].message.content === "" || $json.choices[0].message.content === undefined }}
```

**Stop and Error node** — use this to formally mark the execution as failed after alerting:

- Set **Error Message** to: `AI response was empty. Manual review required.`

---

## Failed API Call Handling

### Pattern: Continue on Fail + Downstream Check

Enable **Continue on Fail** on the API node, then add an **IF** node immediately after to check whether the previous node errored:

**IF node condition:**

```
{{ $json.error !== undefined }}
```

- **True branch:** Handle the error (alert, log, fallback value)
- **False branch:** Continue with the successful response

### Pattern: Fallback Value on Error

Use a **Code** node after a failing node (with Continue on Fail enabled) to provide a safe default:

```javascript
// Code node: provide fallback if previous node errored
const input = $input.first().json;

if (input.error) {
  return [{
    json: {
      result: null,
      error: true,
      errorMessage: input.error.message,
      fallback: "Unable to process at this time. Please retry manually."
    }
  }];
}

return [{ json: { result: input, error: false } }];
```

---

## Missing Field Validation with IF Nodes

Validate required fields at the start of every webhook-triggered workflow before passing data to AI nodes or external APIs.

### Required Fields Validation Pattern

Add an **IF** node as the first step after the Webhook node:

**Condition (check all required fields exist):**

```
{{ !$json.email || !$json.name || !$json.message }}
```

- **True (missing fields):** Return an error response immediately
- **False (all fields present):** Continue processing

**Respond to Webhook node on the true branch:**

```json
{
  "success": false,
  "error": "Missing required fields: email, name, and message are required."
}
```

### Type Validation with Code Node

For more complex validation, use a Code node:

```javascript
const data = $input.first().json;
const errors = [];

if (!data.email || !data.email.includes('@')) {
  errors.push('Invalid or missing email address');
}
if (!data.name || data.name.trim().length < 2) {
  errors.push('Name must be at least 2 characters');
}
if (typeof data.budget !== 'number' || data.budget < 0) {
  errors.push('Budget must be a non-negative number');
}

if (errors.length > 0) {
  return [{
    json: {
      valid: false,
      errors: errors
    }
  }];
}

return [{ json: { valid: true, data: data } }];
```

---

## OpenAI Failure Handling

### Rate Limit Errors (429)

OpenAI returns HTTP 429 when you exceed your rate limit or quota.

**Node configuration:**

- Enable **Retry on Fail**
- Set **Max Tries**: 5
- Set **Wait Between Tries**: 10000 (10 seconds)

**Additional handling with Code node** (after OpenAI, with Continue on Fail enabled):

```javascript
const response = $input.first().json;

// Check for rate limit error
if (response.error && response.error.code === 'rate_limit_exceeded') {
  // Log and return fallback
  return [{
    json: {
      aiResult: null,
      skipped: true,
      skipReason: 'OpenAI rate limit reached. Will retry in next scheduled run.',
      timestamp: new Date().toISOString()
    }
  }];
}

return [{ json: response }];
```

### Timeout Errors

For workflows with long prompts or large inputs, set an appropriate timeout in the HTTP Request node (if calling OpenAI via HTTP Request instead of the built-in node):

```json
{
  "timeout": 60000
}
```

For the built-in OpenAI node, set `N8N_EXECUTION_TIMEOUT` in your environment to allow long executions.

### Malformed or Empty Responses

Always validate the OpenAI response structure before using it downstream:

```javascript
const response = $input.first().json;

// Validate response structure
if (!response.choices || response.choices.length === 0) {
  throw new Error('OpenAI returned no choices in response');
}

const content = response.choices[0]?.message?.content;

if (!content || content.trim() === '') {
  throw new Error('OpenAI returned an empty response');
}

return [{ json: { content: content } }];
```

---

## JSON Parsing Error Handling

AI models sometimes return text that is almost-but-not-quite valid JSON. Handle this defensively.

### Safe JSON Parse with Code Node

```javascript
const response = $input.first().json;
const rawContent = response.choices[0].message.content;

let parsed;

try {
  // First attempt: direct parse
  parsed = JSON.parse(rawContent);
} catch (firstError) {
  try {
    // Second attempt: extract JSON from markdown code block
    const match = rawContent.match(/```(?:json)?\s*([\s\S]*?)```/);
    if (match) {
      parsed = JSON.parse(match[1].trim());
    } else {
      throw new Error('No JSON code block found');
    }
  } catch (secondError) {
    try {
      // Third attempt: find first { ... } block
      const jsonStart = rawContent.indexOf('{');
      const jsonEnd = rawContent.lastIndexOf('}');
      if (jsonStart !== -1 && jsonEnd !== -1) {
        parsed = JSON.parse(rawContent.substring(jsonStart, jsonEnd + 1));
      } else {
        throw new Error('No JSON object found in response');
      }
    } catch (thirdError) {
      // All attempts failed — return error state
      return [{
        json: {
          parseError: true,
          rawContent: rawContent,
          errorMessage: thirdError.message
        }
      }];
    }
  }
}

return [{ json: parsed }];
```

### Handling Parse Failures Downstream

After the safe parse node, add an IF node:

**Condition:**

```
{{ $json.parseError === true }}
```

- **True:** Send to manual review queue (Slack alert + log to sheet)
- **False:** Continue with parsed data

---

## Summary: Error Handling Checklist

- [ ] Error Trigger workflow created and assigned to all production workflows
- [ ] Retry on Fail enabled for all external API nodes
- [ ] Required field validation IF node at the start of all webhook workflows
- [ ] OpenAI node has retry enabled with adequate wait time
- [ ] JSON parsing uses try/catch with fallback logic
- [ ] Continue on Fail enabled on nodes where partial failure is acceptable
- [ ] Slack alert connected to the Error Trigger workflow
- [ ] Error log spreadsheet capturing all failures with timestamps
- [ ] Stop and Error node used when a workflow must not silently continue after failure
