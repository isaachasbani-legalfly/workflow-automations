# Line Items to Company Property — Architecture v2.1

## Overview

This workflow extracts line items from deals that move to Closed Won in the Sales or Expansion pipeline, formats them as a human-readable block (date header, product name, quantity, net price), and appends them to the company's `productpurchased` property in HubSpot.

Additionally, it detects whether the deal includes SSO or Legal Radar products and sets the corresponding company properties (`has_SSO`, `has_legal_radar`) to `"yes"`.

Triggered by a HubSpot internal workflow via webhook whenever a deal closes. Processes all companies regardless of lifecycle stage. Deduplicates on re-fire. Errors route to the shared Slack error handler.

## Workflow Diagram

```mermaid
flowchart TD
    WH["Webhook: Deal Closed Won\nPOST /line-items-closed-won"] --> NW["Normalize Webhook Data\nExtract dealId from body"]
    NW --> GD["Get Deal + Associations\nGET deal props + line_items + companies"]
    GD --> EDD["Extract Deal Data\ndealId, closedate, currency, companyId, lineItemIds\nskip if no company or no line items"]
    EDD --> BLI["Batch Read Line Items\nname, amount, qty, currency"]
    BLI --> FLI["Format Line Items\n[DD/MM/YYYY · Pipeline]\nProduct × Qty — €Price\n+ detect hasSSO / hasLegalRadar"]
    FLI --> GBC["Group by Company\nMerge deals per company\nOR product flags across deals"]
    GBC --> GC["Get Company Current Value\nproductpurchased, name"]
    GC --> AP["Append & Prepare Update\nDeduplicate + append\nPass through product flags"]
    AP --> UC["Update Company\nPATCH productpurchased\n+ has_SSO + has_legal_radar"]

    style WH fill:#e1f5fe
    style NW fill:#e8eaf6
    style GD fill:#bbdefb
    style EDD fill:#e8eaf6
    style BLI fill:#bbdefb
    style FLI fill:#e8eaf6
    style GBC fill:#e8eaf6
    style GC fill:#bbdefb
    style AP fill:#e8eaf6
    style UC fill:#bbdefb
```

## Node Reference

### Webhook: Deal Closed Won (`webhook-1`)
- **Type**: n8n-nodes-base.webhook v2.1
- **Purpose**: Production entry point — receives deal ID from HubSpot internal workflow when a deal moves to Closed Won
- **Config**: POST method, path `line-items-closed-won`
- **Output**: `{body: {dealId: "123"}}`

### Normalize Webhook Data (`normalize-webhook`)
- **Type**: n8n-nodes-base.code v2
- **Purpose**: Extracts dealId from webhook body, handles both `body.dealId` and root `dealId` formats
- **Mode**: Run Once for All Items
- **Error**: Throws if no dealId found
- **Output**: `{dealId: "123"}`

### Get Deal + Associations (`get-deal`)
- **Type**: n8n-nodes-base.httpRequest v4.4
- **Purpose**: Fetches deal properties AND associated line item/company IDs in one API call
- **API**: GET `/crm/v3/objects/deals/{dealId}?properties=pipeline,deal_currency_code,closedate&associations=line_items,companies`
- **Retry**: 3 attempts, 1s between tries

### Extract Deal Data (`extract-deal-data`)
- **Type**: n8n-nodes-base.code v2
- **Purpose**: Extracts and structures deal metadata; skips deals with no company or no line items
- **Mode**: Run Once for All Items (flatMap)
- **Skip logic**: Returns `[]` (no error) if deal has no associated company or no line items — handles churn deals and deals without products gracefully
- **Handles**: Both `line items` (space) and `line_items` (underscore) association key formats
- **Output**: `{dealId, closedate, currency, pipeline, companyId, lineItemIds: [...]}`

### Batch Read Line Items (`batch-line-items`)
- **Type**: n8n-nodes-base.httpRequest v4.4
- **Purpose**: Fetches all line item details in a single batch call
- **API**: POST `/crm/v3/objects/line_items/batch/read`
- **Properties**: `name`, `amount`, `quantity`, `hs_line_item_currency_code`
- **Body**: Dynamic — built from `$json.lineItemIds`
- **Retry**: 3 attempts, 1s between tries

### Format Line Items (`format-line-items`)
- **Type**: n8n-nodes-base.code v2
- **Purpose**: Formats each deal's line items into a readable block with a date/pipeline header. Also detects product flags (SSO, Legal Radar) from raw line item names.
- **Mode**: Run Once for All Items (flatMap, skips if no results)
- **Data sources**: Line items from `$input`, deal metadata from `$('Extract Deal Data').all()[index]`
- **Currency mapping**: EUR→€, USD→$, GBP→£, CAD→CA$
- **Pipeline mapping**: `default`→New Business, `3585124587`→Expansion
- **Date format**: DD/MM/YYYY
- **Product flag detection**: Case-insensitive substring match on raw line item `name` property:
  - `hasSSO`: name contains `"sso"` (matches e.g. "ACCOUNT Single Sign On (SSO) Per User Seat (Annual)")
  - `hasLegalRadar`: name contains `"legal radar"` (matches e.g. "PRODUCT Legal Radar (Annual)")
- **Output format**:
  ```
  [03/03/2026 · New Business]
  Contract License × 2 — €10,000.00
  Implementation Services × 1 — €3,500.00
  ```
- **Pipeline detection**: `isNewBusiness = true` when pipeline is `default` (Sales Pipeline)
- **Output**: `{companyId, newLineItems: "formatted string", hasSSO: boolean, hasLegalRadar: boolean, isNewBusiness: boolean}`

### Group by Company (`group-by-company`)
- **Type**: n8n-nodes-base.code v2
- **Purpose**: Merges multiple deal blocks for the same company. ORs product flags and `isNewBusiness` across deals — if any deal for a company has SSO or Legal Radar, the flag is `true`. If any deal is New Business, `isNewBusiness` is `true`.
- **Mode**: Run Once for All Items
- **Join**: Multiple deal blocks separated by `\n\n` (blank line)
- **Output**: `{companyId, newLineItems: "merged string", hasSSO: boolean, hasLegalRadar: boolean, isNewBusiness: boolean}`

### Get Company Current Value (`get-company`)
- **Type**: n8n-nodes-base.httpRequest v4.4
- **Purpose**: Reads the company's current `productpurchased` value before appending
- **API**: GET `/crm/v3/objects/companies/{companyId}?properties=productpurchased,name,lifecyclestage`
- **Retry**: 3 attempts, 1s between tries

### Append & Prepare Update (`append-prepare`)
- **Type**: n8n-nodes-base.code v2
- **Purpose**: Combines existing property text with new line items; deduplicates to prevent double-writes. Passes through product flags from Group by Company.
- **Mode**: Run Once for All Items
- **Data sources**: Company data from `$input`, grouped line items from `$('Group by Company').all()[i]`
- **Deduplication**: Splits existing value by `\n`; only adds lines not already present. Empty strings (`""`) are always passed through to preserve blank line separators between deal groups.
- **Append separator**: Uses `\n\n` between old content and newly appended content
- **Output**: `{companyId, companyName, updatedProductPurchased: "full text", hasSSO: boolean, hasLegalRadar: boolean, isNewBusiness: boolean}`

### Update Company (`update-company`)
- **Type**: n8n-nodes-base.httpRequest v4.4
- **Purpose**: Writes the updated `productpurchased` property to the company. Sets `has_sso` and `has_legal_radar` based on pipeline-aware logic.
- **API**: PATCH `/crm/v3/objects/companies/{companyId}`
- **Body**: `{properties: {productpurchased: "...", ...has_sso?, ...has_legal_radar?}}`
- **Pipeline-aware logic**:
  - **New Business** (`isNewBusiness = true`): Always sets both properties explicitly — `has_sso: "Yes"/"No"` and `has_legal_radar: "true"/"false"`. New clients get a complete product profile.
  - **Expansion** (`isNewBusiness = false`): Only includes `has_sso: "Yes"` or `has_legal_radar: "true"` when the product is found. When not found, the property is omitted from the PATCH — preserving any existing "Yes"/"true" from a prior deal.
- **HubSpot property formats** (enumeration dropdowns):
  - `has_sso`: values `"Yes"` / `"No"` (label: Yes / No)
  - `has_legal_radar`: values `"true"` / `"false"` (label: Yes / No)
- **Retry**: 3 attempts, 1s between tries

## Output Format

```
[03/03/2026 · New Business]
Contract License × 2 — €10,000.00
Implementation Services × 1 — €3,500.00

[15/01/2026 · Expansion]
Platform Subscription × 1 — €24,000.00
```

Each deal group is separated by a blank line. New groups are appended below existing content, also separated by a blank line.

## Product Flag Behavior

| Line item name contains | Company property | HubSpot type | Yes value | No value |
|------------------------|-----------------|-------------|-----------|----------|
| `"sso"` (case-insensitive) | `has_sso` | enumeration dropdown | `"Yes"` | `"No"` |
| `"legal radar"` (case-insensitive) | `has_legal_radar` | enumeration dropdown | `"true"` | `"false"` |

### Pipeline-aware logic

| Pipeline | Product found | Action |
|----------|--------------|--------|
| **New Business** | Yes | Set to Yes/true |
| **New Business** | No | Set to No/false |
| **Expansion** | Yes | Set to Yes/true |
| **Expansion** | No | Not touched (preserves existing value) |

- **New Business deals** set both flags explicitly because we know the full product set — a new client without SSO should have `has_sso = No`
- **Expansion deals** only upgrade flags to Yes — they never downgrade an existing Yes back to No, because the company may have acquired the product in a prior deal
- **Idempotent**: Writing `"Yes"` when already `"Yes"` is harmless (HubSpot no-ops)

## Error Handling

- All HTTP Request nodes have **retry on fail** (3 attempts, 1s wait)
- Deals with no line items or no associated company are **silently skipped** (no error thrown)
- Webhook responds immediately — processing is asynchronous
- Workflow errors route to **Error Handler - Slack Notification** (`TA6Iq4wMW0KYsCiH`) via `errorWorkflow` setting

## Design Decisions

1. **Webhook + HubSpot internal workflow (not HubSpot Trigger node)**: The n8n HubSpot Trigger fires on ANY deal stage change. A HubSpot internal workflow filters at the source — only Closed Won in the right pipelines triggers n8n.

2. **No lifecycle stage filter**: All companies are updated regardless of lifecycle stage. New clients signing their first deal need the property populated too — filtering by `customer` would miss them.

3. **Skip instead of throw for missing data**: Churn deals and deals without line items return `[]` from Extract Deal Data rather than throwing. This keeps the workflow running cleanly without noise in the error handler.

4. **`amount` field (not `price`)**: The `amount` property is the net price after discounts — exactly what the client paid. `price` is the unit list price before discounts.

5. **Deduplication on every run**: If the HubSpot workflow fires the webhook twice for the same deal, no duplicate lines are written. The check compares full line strings.

6. **Blank line deduplication exception**: Empty strings (`""`) are always passed through the deduplication filter, because they're structural separators between deal groups — not content lines. Filtering them would collapse multiple deal groups into one block.

7. **`\n\n` separator when appending**: New content is joined to existing content with a double newline, ensuring a blank line separator between old and new deal groups regardless of which batch they came from.

8. **Single GET with associations**: The `?associations=line_items,companies` parameter fetches deal properties + association IDs in one API call, reducing latency and API quota usage.

9. **Batch read for line items**: A deal may have multiple line items. The batch read endpoint handles all of them in one call.

10. **Product flag detection at Format Line Items**: Detection happens here because we already have the raw line item names. No extra API call needed. Flags flow through Group by Company (OR logic) → Append & Prepare → Update Company (same PATCH call).

11. **Case-insensitive substring matching for product flags**: More resilient than exact name matching — survives minor naming changes (e.g., "SSO" vs "Sso", "Legal Radar" vs "LEGAL RADAR"). Substring match on `"sso"` and `"legal radar"` is specific enough to avoid false positives from other product names.

12. **Pipeline-aware product flag logic**: New Business deals set both `has_sso` and `has_legal_radar` explicitly (Yes/No or true/false) because we know the full product set for a new client. Expansion deals only set flags to Yes when the product is found — they never downgrade an existing Yes, because the company may have acquired the product in a prior deal.

13. **Different enumeration values per property**: `has_sso` uses `"Yes"`/`"No"` while `has_legal_radar` uses `"true"`/`"false"` — both are HubSpot enumeration dropdowns, but they were created with different value formats. The workflow respects each property's actual enum values.

## Credentials Required

| Service | Credential Name | Type | Used By |
|---------|----------------|------|---------|
| HubSpot | hubspot | App Token (Private App) | All HTTP Request nodes |

## n8n Instance

- **Workflow ID**: `TQHGk5e2V0XL8D4f`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TQHGk5e2V0XL8D4f
- **Error Handler**: `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
