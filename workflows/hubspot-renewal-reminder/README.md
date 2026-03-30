# HubSpot Monthly Renewal Reminder

On the 1st of each month, this workflow sends a Slack digest of all customer companies with a contract renewal date in that month. Each company is listed with its name, exact renewal date, and a direct link to the HubSpot record.

---

## Why it exists

Renewal dates can slip under the radar when they're buried in individual HubSpot records. This workflow surfaces them proactively in Slack at the start of each month so the team knows exactly which contracts are coming up.

---

## When it runs

**1st of every month at 8:00 AM UTC.**

---

## What it does, step by step

### 1. Calculate the current month's date range
Computes the first day of the current month and the first day of the next month (UTC timestamps) to use as HubSpot search filters.

### 2. Fetch companies renewing this month
Queries HubSpot for all companies where:
- `contract___renewal_date` is within the current month (GTE first day, LT next month)
- `lifecyclestage` = `customer`

### 3. Check for results
If no companies are renewing this month, the workflow ends silently.

### 4. Format and send Slack message
Builds a digest sorted by renewal date, listing each company with its name, formatted date, and HubSpot link. Posts to the configured Slack channel.

---

## Slack message format

```
:calendar: Renewals for April 2026

5 companies renewing this month:

- Acme Corp -- April 5, 2026 -- View
- Beta Inc -- April 12, 2026 -- View
- Gamma Ltd -- April 18, 2026 -- View
```

---

## Credentials required

| Service | What it's used for |
|---------|--------------------|
| HubSpot | Read company data (renewal dates) |
| Slack | Post monthly reminder to channel `C0APNNQAN5B` |

---

## Error handling

Linked to the shared error handler workflow (`TA6Iq4wMW0KYsCiH`). If the workflow crashes, an alert is automatically sent to the errors Slack channel.

---

## Files in this folder

| File | Purpose |
|------|---------|
| `workflow-v1.0.json` | The n8n workflow export — source of truth |
| `ARCHITECTURE-v1.0.md` | Full technical reference: all nodes, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `CHANGELOG.md` | Version history |

---

## n8n instance

**Workflow ID**: `BKMGY6W4ZHwawdeG`
**URL**: [https://legalfly.app.n8n.cloud/workflow/BKMGY6W4ZHwawdeG](https://legalfly.app.n8n.cloud/workflow/BKMGY6W4ZHwawdeG)
