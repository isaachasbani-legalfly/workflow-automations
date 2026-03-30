# HubSpot Days Until Renewal Calculator

This workflow automatically calculates the number of days until each company's next contract renewal and writes the result to HubSpot. It runs every night, fetching all customer companies that have a renewal date set, computing the difference from today, and batch-updating the `days_until_next_renewal` property. Companies past their renewal date are marked as "Overdue".

---

## Why it exists

The `days_until_next_renewal` field needs to be recalculated daily to stay accurate. Doing this manually is impractical. This workflow handles it silently overnight so the CRM always has fresh data for reporting, filtering, and alerting.

---

## When it runs

Every day at **12:05 AM UTC**.

---

## What it does, step by step

### 1. Fetch all customer companies with a renewal date (with pagination)
Queries HubSpot for all companies where `lifecyclestage = customer` AND `contract___renewal_date` is set. Uses cursor-based pagination (200 per page) to ensure every company is captured regardless of volume.

### 2. Calculate days until renewal
For each company:
- Parses the `contract___renewal_date` value
- Calculates `days = ceil((renewal_date - today_midnight_UTC) / 86400000)`
- If days >= 0: writes the exact number as a string (e.g. `"45"`)
- If days < 0: writes `"Overdue"`

### 3. Batch update HubSpot
Groups companies into batches of 100 and writes `days_until_next_renewal` using the HubSpot Batch Update API for speed.

---

## HubSpot properties

| Property | Type | Purpose |
|----------|------|---------|
| `contract___renewal_date` | date | Source: the company's next renewal date |
| `days_until_next_renewal` | string | Target: calculated days or "Overdue" |
| `lifecyclestage` | enumeration | Filter: only processes companies with value `customer` |

---

## Credentials required

| Service | What it's used for |
|---------|--------------------|
| HubSpot | Read company data, batch update `days_until_next_renewal` |

---

## Error handling

Linked to the shared error handler workflow (`TA6Iq4wMW0KYsCiH`). If the workflow crashes, an alert is automatically sent to the errors Slack channel.

---

## Files in this folder

| File | Purpose |
|------|---------|
| `workflow-v1.0.json` | The n8n workflow export — source of truth |
| `ARCHITECTURE-v1.0.md` | Full technical reference: all nodes, routing logic, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `CHANGELOG.md` | Version history |

---

## n8n instance

**Workflow ID**: `CXKaD0HuoP9G5wDM`
**URL**: [https://legalfly.app.n8n.cloud/workflow/CXKaD0HuoP9G5wDM](https://legalfly.app.n8n.cloud/workflow/CXKaD0HuoP9G5wDM)
