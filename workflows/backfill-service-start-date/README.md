# Backfill Company Service Start Date

One-time manual workflow that sets each customer company's **Contract - Latest Service Start Date** (`contract___service_start_date`) to the earliest `service_start_date` from their associated closed-won deals.

## Trigger

Manual — run once from the n8n editor.

## How It Works

1. **Search All Customers** — Fetches all companies with `lifecyclestage = customer` from HubSpot (currently 140).
2. **Extract Companies** — Flattens the search results into individual items with `companyId` and `companyName`.
3. **SplitInBatches** — Processes one company at a time to respect HubSpot rate limits.
4. **Get Deal Associations** — For each company, retrieves associated deal IDs via HubSpot v4 Associations API.
5. **Has Deals?** — Checks if the company has any associated deals. If not, skips to the next company.
6. **Batch Read Deals** — Reads all associated deals in one call, fetching `service_start_date`, `dealstage`, and `dealname`.
7. **Find Earliest Service Date** — Filters for `closedwon` deals that have a `service_start_date`, sorts ascending, and picks the earliest.
8. **Has Date?** — Checks if a qualifying deal was found. If not, skips.
9. **Update Company** — Sets the company's `contract___service_start_date` to the earliest deal service start date.

## What Gets Skipped

- Companies with no associated deals
- Companies where no closed-won deals have a `service_start_date`
- The `closedate` field is **never** used as a fallback

## Credentials

| Service  | Credential Name | Type             | Used For               |
|----------|----------------|------------------|------------------------|
| HubSpot  | hubspot        | hubspotAppToken  | All HTTP Request nodes |

## n8n Instance

- **Workflow ID**: `HJGuurVOZ5Elh1xm`
- **URL**: https://legalfly.app.n8n.cloud/workflow/HJGuurVOZ5Elh1xm
- **Error Workflow**: `TA6Iq4wMW0KYsCiH`
