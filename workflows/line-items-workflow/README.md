# Line Items to Company Property

Automatically captures what products a company purchased, at what price, and when — by extracting line items from closed-won deals and appending them to a company property in HubSpot. Also detects SSO and Legal Radar products and sets the corresponding company flags (`has_sso`, `has_legal_radar`).

## Why it exists

When working on expansion deals, the sales team needs to quickly see what products a company already purchased and at what price. This workflow eliminates the need to dig through past deals by maintaining a running log on the company record itself.

## When it runs

Triggered via webhook when a HubSpot internal workflow detects a deal moving to **Closed Won** in the **Sales Pipeline** or **Expansion Pipeline**.

## What it does, step by step

1. **Webhook** receives a POST with `{"dealId": "<id>"}` from a HubSpot internal workflow when a deal moves to Closed Won
2. **Normalize Webhook Data** extracts the deal ID from the payload (handles both `body.dealId` and root `dealId` formats)
3. **Get Deal + Associations** fetches deal properties (pipeline, currency, close date) and associated line item IDs + company ID in a single API call
4. **Extract Deal Data** structures the deal metadata (deal ID, close date, currency, pipeline, company ID, line item IDs). Silently skips deals with no line items or no associated company — no error thrown
5. **Batch Read Line Items** fetches all line item details (name, net price after discounts, quantity, currency) in one batch call
6. **Format Line Items** formats each deal's line items into a readable block with a date/pipeline header:
   ```
   [03/03/2026 · New Business]
   Contract License × 2 — €10,000.00
   Implementation Services × 1 — €3,500.00
   ```
   Also scans the raw line item names (case-insensitive substring match) to detect product flags:
   - Line item name contains **"sso"** → `hasSSO = true`
   - Line item name contains **"legal radar"** → `hasLegalRadar = true`
   - Pipeline is Sales Pipeline (`default`) → `isNewBusiness = true`
7. **Group by Company** merges multiple deal blocks for the same company (separated by blank lines). ORs all flags across deals — if any deal has the product, the flag is true
8. **Get Company Current Value** reads the company's existing `productpurchased` property from HubSpot
9. **Append & Prepare Update** appends new line item content to the existing value with deduplication (prevents double-writes if the webhook fires twice for the same deal). Passes through all product flags and the pipeline context
10. **Update Company** writes the updated properties back to HubSpot in a single PATCH call:
    - `productpurchased` — always updated with the formatted line items text
    - `has_sso` and `has_legal_radar` — set based on pipeline type:

    | Pipeline | Product found | Action |
    |----------|--------------|--------|
    | **New Business** | Yes | Set to Yes/true |
    | **New Business** | No | Set to No/false |
    | **Expansion** | Yes | Set to Yes/true |
    | **Expansion** | No | Not touched (preserves existing value) |

    This ensures new clients get both flags set explicitly, while expansion deals only upgrade flags to Yes — never downgrade an existing Yes back to No.

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
| `ARCHITECTURE-v2.1.md` | Full technical reference: nodes, routing, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `workflow-v2.1.json` | n8n workflow export — source of truth |
| `CHANGELOG.md` | Version history |

## n8n Instance

- **Workflow ID**: `TQHGk5e2V0XL8D4f`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TQHGk5e2V0XL8D4f
