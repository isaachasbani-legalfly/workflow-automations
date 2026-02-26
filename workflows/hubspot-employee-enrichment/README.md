# HubSpot Employee Count Enrichment

Nightly workflow that enriches HubSpot companies with employee counts using a multi-stage pipeline: Gemini classifies companies as independent vs subsidiary, Amplemarket provides batch employee data (using parent domains for subsidiaries), and Jina+Gemini estimates counts as a fallback.

## Current Version

**v2.0** — Complete redesign with batch Amplemarket, upfront subsidiary classification, and scrape fallback.
- v1.0 (`u9IcVLMFzBO6Idkw`) kept as rollback.

## Trigger

**Schedule**: Daily at 02:01 Europe/London

## How It Works

1. **Paginated fetch** — ALL HubSpot companies created yesterday (cursor-based, 200/page)
2. **Gemini batch classification** — Single API call classifies all companies as independent or subsidiary/branch
3. **3-way route**:
   - (A) No employee count → needs enrichment
   - (B) Has count BUT is subsidiary → needs re-enrichment with parent data
   - (C) Has count AND is independent → skip
4. **Batch Amplemarket** — Single POST with all companies needing enrichment, poll until complete
   - For subsidiaries: uses parent domain instead of branch domain
5. **Scrape fallback** — Unresolved companies with a domain: Jina scrape → Gemini estimate
6. **Default** — No domain or empty scrape: assign 5 employees
7. **Write back** to HubSpot: `numberofemployees`, enrichment source, `is_subsidiary`, `parent_company_name`
8. **Slack summary** with categorized results

## Data Flow

```
Schedule (02:01 daily)
  → Paginated HubSpot fetch (ALL companies created yesterday)
  → Gemini batch: classify ALL as independent vs subsidiary
  → 3-way split:
      (A) No employee count → needs enrichment
      (B) Has count + subsidiary → re-enrich with parent data
      (C) Has count + independent → skip
  → For (A)+(B): replace branch domains with parent domain
  → Batch Amplemarket enrichment (POST + poll loop)
  → Unresolved + has domain: Jina scrape → Gemini estimate
  → Unresolved + no domain: default = 5
  → Update HubSpot (employee count + source + is_subsidiary + parent_company_name)
  → Slack summary
```

## Enrichment Source Values

| Value | Meaning |
|-------|---------|
| `Amplemarket` | Direct Amplemarket data, independent company |
| `Amplemarket (parent: {name})` | Subsidiary; parent's Amplemarket data used |
| `Estimated` | Gemini estimated from website scrape, or default (5) |

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| HubSpot | hubspot | App Token | Fetch + update companies |
| Amplemarket | amplemarket | HTTP Header Auth | Batch employee enrichment |
| Google Gemini | Gemini | Google Palm API | Classification + estimation |
| Slack | Slack | Slack API | Summary notifications |

## HubSpot Properties

| Property | Internal Name | Type | Notes |
|----------|--------------|------|-------|
| Employee Count | `numberofemployees` | Standard | Number |
| Enrichment Source | `number-employees-enrichment-source` | Custom | Text (existing) |
| Is Subsidiary | `is_subsidiary` | Custom | Checkbox (NEW — create in HubSpot) |
| Parent Company | `parent_company_name` | Custom | Text (NEW — create in HubSpot) |

## n8n Instance

- **Workflow ID**: `TxZMblqjvC86tHAu`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TxZMblqjvC86tHAu
- **Error Workflow**: `TA6Iq4wMW0KYsCiH` (Error Handler — Slack Notification)
- **Status**: Inactive (activate after testing)
- **v1.0 (rollback)**: `u9IcVLMFzBO6Idkw`
