# Line Items to Company Property

Automatically captures what products a company purchased, at what price, and when — by extracting line items from closed-won deals and appending them to a company property in HubSpot.

## Why it exists

When working on expansion deals, the sales team needs to quickly see what products a company already purchased and at what price. This workflow eliminates the need to dig through past deals by maintaining a running log on the company record itself.

## When it runs

Triggered via webhook when a HubSpot internal workflow detects a deal moving to **Closed Won** in the **Sales Pipeline** or **Expansion Pipeline**.

## What it does, step by step

1. **Webhook** receives a POST with `{"dealId": "<id>"}` from a HubSpot internal workflow
2. **Normalize** extracts the deal ID from the payload
3. **Get Deal + Associations** — fetches deal properties (pipeline, currency, close date) and associated line item IDs + company ID in a single API call
4. **Extract Deal Data** — structures the deal metadata; silently skips deals with no line items or no associated company
5. **Batch Read Line Items** — fetches all line item details (name, net price after discounts, quantity, currency) in one batch call
6. **Format Line Items** — formats each deal as:
   ```
   [03/03/2026 · New Business]
   Contract License × 2 — €10,000.00
   Implementation Services × 1 — €3,500.00
   ```
7. **Group by Company** — merges multiple deal blocks for the same company (separated by blank line)
8. **Get Company Current Value** — reads the company's existing `productpurchased` property
9. **Append & Prepare** — appends new content to existing value with deduplication (prevents double-writes if webhook fires twice)
10. **Update Company** — writes the updated `productpurchased` value back to HubSpot

## Output Format

```
[03/03/2026 · New Business]
Contract License × 2 — €10,000.00
Implementation Services × 1 — €3,500.00

[15/01/2026 · Expansion]
Platform Subscription × 1 — €24,000.00
```

Each deal group has a `[Date · Pipeline]` header. Multiple deals are separated by blank lines. New entries are appended below existing content.

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

A HubSpot workflow must send a webhook to n8n when a deal closes:

1. **Trigger**: Deal enters stage `07_Closed Won`
2. **Filter**: Pipeline is `Sales Pipeline` OR `Expansion Pipeline`
3. **Action**: Send webhook POST to `https://legalfly.app.n8n.cloud/webhook/line-items-closed-won`
4. **Body**: `{"dealId": "{{deal.id}}"}`

**Important**: Set the HubSpot workflow to not re-enroll historical deals — use an enrollment filter like `Deal close date is after [go-live date]` to avoid reprocessing already-backfilled deals.

## Error Handling

Workflow errors send a Slack alert via the shared **Error Handler - Slack Notification** workflow (`TA6Iq4wMW0KYsCiH`).

## Files in this folder

| File | Purpose |
|------|---------|
| `README.md` | This file — overview, setup, credentials |
| `ARCHITECTURE-v2.0.md` | Full technical reference: nodes, routing, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `workflow-v2.0.json` | n8n workflow export — source of truth |
| `CHANGELOG.md` | Version history |

## n8n Instance

- **Workflow ID**: `TQHGk5e2V0XL8D4f`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TQHGk5e2V0XL8D4f
