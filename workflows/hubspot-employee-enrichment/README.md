# HubSpot Employee Count Enrichment

Two-track nightly workflow that enriches HubSpot companies:

- **Track A**: Enrich `numberofemployees` for companies with no value, using Amplemarket. Default to 5 if no data.
- **Track B**: For subsidiaries, get parent/group-level employee count and write to `group_number_of_employees`.

Gemini classifies companies as independent vs subsidiary with a two-tier confidence system (HIGH auto-write, MEDIUM not written + flagged in Slack for review). `numberofemployees` is NEVER overwritten by Track B.

## Current Version

**v4.0** -- Two-track enrichment, confidence tiers, self-referencing block, no domain-based classification rules.
- v3.0 version history preserved in n8n as rollback.
- v1.0 (`u9IcVLMFzBO6Idkw`) kept as rollback.

## Trigger

**Schedule**: Daily at 02:01 Europe/London

## How It Works

1. **Paginated fetch** -- ALL HubSpot companies created yesterday (cursor-based, 200/page)
2. **Gemini batch classification** -- Single API call (gemini-2.5-pro) classifies all companies as independent or subsidiary with confidence (HIGH/MEDIUM)
3. **Self-referencing block** -- Prevents a company from being classified as subsidiary of itself
4. **Parent company lookup** -- Searches HubSpot for parent companies; sets `parent_company_name` as clickable HubSpot URL if found
5. **Route**: companies needing enrichment (no employee count OR is subsidiary) go to Amplemarket; others skip
6. **Combined Amplemarket batch** -- Single POST with all unique domains (company domains for Track A + parent domains for Track B)
7. **Smart merge** -- Maps Amplemarket results to the right property per company:
   - Track A: `numberofemployees` from company's own domain (or default 5)
   - Track B: `group_number_of_employees` from parent domain
8. **Write back** to HubSpot (conditional -- only writes fields that need updating)
9. **Slack summary** with Track A + Track B sections, confidence flags, and review section for medium-confidence items

## Data Flow

```
Schedule (02:01 daily)
  -> Paginated HubSpot fetch (ALL companies created yesterday)
  -> Gemini batch (gemini-2.5-pro): classify with confidence tiers
  -> Self-referencing block (parent != self)
  -> Parent company HubSpot lookup (clickable URL if found)
  -> Route: no count OR subsidiary -> needs enrichment
  -> Combined Amplemarket batch (company + parent domains, de-duped)
  -> Smart merge:
      Track A: own domain -> numberofemployees (or default 5)
      Track B: parent domain -> group_number_of_employees
  -> Conditional HubSpot update (only writes what's needed)
  -> Slack summary (Track A + Track B + review section)
```

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| HubSpot | hubspot | App Token | Fetch + update companies |
| Amplemarket | amplemarket | HTTP Header Auth | Batch employee enrichment |
| Google Gemini | Gemini | Google Palm API | Classification only |
| Slack | Slack | Slack API | Summary notifications |

## HubSpot Properties

| Property | Internal Name | Type | Track | Notes |
|----------|--------------|------|-------|-------|
| Employee Count | `numberofemployees` | Standard | A | Only written when no existing value |
| Enrichment Source | `number-employees-enrichment-source` | Custom | A | `Amplemarket` or `Estimated` |
| Is Subsidiary | `is_subsidiary` | Custom | B | Checkbox |
| Parent Company | `parent_company_name` | Custom | B | Clickable HubSpot URL if parent found |
| Group Employee Count | `group_number_of_employees` | Custom | B | Parent/group-level count |

## n8n Instance

- **Workflow ID**: `TxZMblqjvC86tHAu`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TxZMblqjvC86tHAu
- **Error Workflow**: `TA6Iq4wMW0KYsCiH` (Error Handler -- Slack Notification)
- **Status**: Inactive (Update HubSpot node disabled during testing)
- **v1.0 (rollback)**: `u9IcVLMFzBO6Idkw`
