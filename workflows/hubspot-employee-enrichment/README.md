# HubSpot Employee Count Enrichment

Nightly workflow that enriches HubSpot companies with employee counts using a two-stage pipeline: Gemini classifies companies as independent vs subsidiary, then Amplemarket provides batch employee data (using parent domains for subsidiaries). Companies Amplemarket can't resolve get a default of 5.

## Current Version

**v3.0** — Simplified: removed scrape fallback (6 nodes), rewrote classification prompt for 5-10% subsidiary rate.
- v2.1 version history preserved in n8n as rollback.
- v1.0 (`u9IcVLMFzBO6Idkw`) kept as rollback.

## Trigger

**Schedule**: Daily at 02:01 Europe/London

## How It Works

1. **Paginated fetch** — ALL HubSpot companies created yesterday (cursor-based, 200/page)
2. **Gemini batch classification** — Single API call (gemini-2.5-pro) classifies all companies as independent or subsidiary/branch with strict rules (6 evidence categories only)
3. **Parent company lookup** — Searches HubSpot for parent companies; sets `parent_company_name` as clickable HubSpot URL if found
4. **3-way route**:
   - (A) No employee count → needs enrichment
   - (B) Has count BUT is subsidiary → needs re-enrichment with parent data
   - (C) Has count AND is independent → skip
5. **Batch Amplemarket** — Single POST with all companies needing enrichment, poll until complete
   - For subsidiaries: uses parent domain instead of branch domain
6. **Default** — Amplemarket has no data: assign 5 employees (tagged as "Estimated")
7. **Write back** to HubSpot: `numberofemployees`, enrichment source, `is_subsidiary`, `parent_company_name`
8. **Slack summary** with 3 categories: Amplemarket, Parent enriched, Default

## Data Flow

```
Schedule (02:01 daily)
  → Paginated HubSpot fetch (ALL companies created yesterday)
  → Gemini batch (gemini-2.5-pro): classify ALL as independent vs subsidiary
  → Parent company HubSpot lookup (clickable URL if found)
  → 3-way split:
      (A) No employee count → needs enrichment
      (B) Has count + subsidiary → re-enrich with parent data
      (C) Has count + independent → skip
  → For (A)+(B): replace branch domains with parent domain
  → Batch Amplemarket enrichment (POST + poll loop)
  → Amplemarket has data → use it. Doesn't → default to 5. Done.
  → Update HubSpot (employee count + source + is_subsidiary + parent_company_name)
  → Slack summary (3 categories)
```

## Enrichment Source Values

| Value | Meaning |
|-------|---------|
| `Amplemarket` | Direct Amplemarket data, independent company |
| `Amplemarket (parent: {name})` | Subsidiary; parent's Amplemarket data used |
| `Estimated` | Default value (5); no data available |

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| HubSpot | hubspot | App Token | Fetch + update companies |
| Amplemarket | amplemarket | HTTP Header Auth | Batch employee enrichment |
| Google Gemini | Gemini | Google Palm API | Classification only |
| Slack | Slack | Slack API | Summary notifications |

## HubSpot Properties

| Property | Internal Name | Type | Notes |
|----------|--------------|------|-------|
| Employee Count | `numberofemployees` | Standard | Number |
| Enrichment Source | `number-employees-enrichment-source` | Custom | Text (existing) |
| Is Subsidiary | `is_subsidiary` | Custom | Checkbox |
| Parent Company | `parent_company_name` | Custom | Text — clickable HubSpot URL if parent found |

## n8n Instance

- **Workflow ID**: `TxZMblqjvC86tHAu`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TxZMblqjvC86tHAu
- **Error Workflow**: `TA6Iq4wMW0KYsCiH` (Error Handler — Slack Notification)
- **Status**: Active
- **v1.0 (rollback)**: `u9IcVLMFzBO6Idkw`
