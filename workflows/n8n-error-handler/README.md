# Error Handler - Slack Notification

A shared error handler workflow that fires a Slack alert whenever a registered n8n workflow crashes with an unhandled error.

---

## How it works

n8n has a built-in concept called an **Error Workflow**: any workflow can nominate a separate workflow to be triggered automatically if it crashes. When a crash happens, n8n calls the Error Workflow with context about what went wrong — which workflow failed, which node was the last one to run, and the error message.

This workflow receives that context and formats it into a readable Slack message.

---

## What triggers it

This workflow is triggered by n8n itself — not by a user or a schedule. It activates when a linked workflow crashes without recovering.

**Key point**: nodes that have "Continue on fail" enabled will silently swallow their own errors and move on. The Error Handler only fires when an error actually crashes the execution (no recovery path exists).

---

## What it does

1. **Error Trigger** — receives the crash context from n8n, including:
   - The name of the failed workflow
   - The last node that ran before the crash
   - The error message
   - A direct link to the failed execution

2. **Format Error Message** — builds a clean Slack message from that context, including the timestamp in London time.

3. **Send Error to Slack** — posts the message to the configured Slack channel (`D0ADELD95GR`).

---

## Slack message format

```
🚨 Workflow Error

Workflow: HubSpot Company Industry Categorization v3.3
Failed Node: Gemini Categorization - HubSpot
Error: Request failed with status 429
Time: 17 Feb 2026, 23:49
View Execution
```

---

## Which workflows use this

| Workflow | ID |
|----------|----|
| HubSpot Company Industry Categorization v3.3 | `8DM3CwXLxOT3G8B7` |
| Line Items to Company Property v2.0 | `TQHGk5e2V0XL8D4f` |

To link this error handler to a new workflow, set the `errorWorkflow` field in that workflow's settings to `TA6Iq4wMW0KYsCiH`.

---

## Credentials required

| Service | What it's used for |
|---------|--------------------|
| Slack | Send the error notification |

---

## n8n instance

**Workflow ID**: `TA6Iq4wMW0KYsCiH`
**URL**: [https://legalfly.app.n8n.cloud/workflow/TA6Iq4wMW0KYsCiH](https://legalfly.app.n8n.cloud/workflow/TA6Iq4wMW0KYsCiH)
