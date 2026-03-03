# Line Items to Company Property

Automatically captures what products a company purchased, at what price, and when — by extracting line items from closed-won deals and appending them to a company property in HubSpot.

## Why it exists

When working on expansion deals, the sales team needs to quickly see what products a company already purchased and at what price. This workflow eliminates the need to dig through past deals by maintaining a running log on the company record itself.

## When it runs

- **Production**: Triggered via webhook when a HubSpot internal workflow detects a deal moving to **Closed Won** in the **Sales Pipeline** or **Expansion Pipeline**
- **Backfill**: Manually triggered to process all existing closed-won deals (up to 100 per run)

## What it does, step by step

### Production Path
1. **Webhook** receives a POST with `{"dealId": "<id>"}` from a HubSpot internal workflow
2. Normalizes the deal ID from the webhook payload

### Backfill Path
1. **Manual Trigger** starts the backfill process
2. **Search** finds all closed-won deals in both Sales and Expansion pipelines
3. Extracts deal IDs from the search results

### Shared Processing (per deal)
4. **Get Deal + Associations** — fetches deal properties (pipeline, currency, close date) and associated line item IDs + company ID in a single API call
5. **Extract Deal Data** — structures the deal metadata, line item IDs, and company ID
6. **Batch Read Line Items** — fetches all line item details (name, net price after discounts, quantity, currency) in one batch call
7. **Format Line Items** — formats each line item as:
   ```
   Product Name - Qty 2 - €1,000.00 - Closed Won on 03/03/2026
   ```
8. **Get Company Current Value** — reads the company's existing `product-purchased` property
9. **Append & Prepare** — appends new line items to existing value (preserving history)
10. **Update Company** — writes the updated `product-purchased` value back to HubSpot

## Output Format

Each line item is formatted as:
```
{Product Name} - Qty {quantity} - {currency symbol}{net price} - Closed Won on {DD/MM/YYYY}
```

Multiple line items are separated by newlines. New entries are appended to existing values.

### Example
```
Contract License - Qty 2 - €10,000.00 - Closed Won on 03/03/2026
Implementation Services - Qty 1 - €3,500.00 - Closed Won on 03/03/2026
Platform Subscription - Qty 1 - €24,000.00 - Closed Won on 15/01/2026
```

## Pipeline & Stage Reference

| Pipeline | Pipeline ID | Closed Won Stage | Stage ID |
|----------|-------------|-----------------|----------|
| Sales Pipeline | `default` | 07_Closed Won | `closedwon` |
| Expansion Pipeline | `3585124587` | 07_Closed Won | `4914500817` |

## Credentials Required

| Service | Credential Type | Credential Name | Used By |
|---------|----------------|-----------------|---------|
| HubSpot | App Token (Private App) | hubspot | All HTTP Request nodes |

## HubSpot Internal Workflow Setup

You need to create a HubSpot workflow that sends a webhook to n8n:

1. **Trigger**: Deal enters stage `07_Closed Won`
2. **Filter**: Pipeline is `Sales Pipeline` OR `Expansion Pipeline`
3. **Action**: Send webhook POST to the n8n webhook URL with body: `{"dealId": "{{deal.id}}"}`

## Files in this folder

| File | Purpose |
|------|---------|
| `README.md` | This file — overview, setup, credentials |
| `ARCHITECTURE-v1.0.md` | Full technical reference: nodes, routing, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `workflow-v1.0.json` | n8n workflow export (source of truth) |
| `CHANGELOG.md` | Version history |

## n8n Instance

- **Workflow ID**: `TQHGk5e2V0XL8D4f`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TQHGk5e2V0XL8D4f
