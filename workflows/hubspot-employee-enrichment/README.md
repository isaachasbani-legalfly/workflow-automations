# HubSpot Employee Count Enrichment

Nightly workflow that enriches HubSpot companies missing `numberofemployees` using Amplemarket data, with Gemini-powered branch/subsidiary detection.

## Trigger

**Schedule**: Daily at 02:00 Europe/London

## How It Works

1. **Fetch** up to 100 HubSpot companies where `numberofemployees` is empty
2. **Lookup** each company via Amplemarket (LinkedIn URL first, then domain fallback)
3. **Branch detection** (Gemini): If employee count found, check if the company is a subsidiary/branch and estimate the parent's global headcount
4. **Write back** to HubSpot: `numberofemployees` + `number-employees-enrichment-source`
5. **Slack summary** with enrichment results

## Data Flow

```
Schedule (02:00)
  -> Fetch companies (numberofemployees = empty, limit 100)
  -> For each company:
       Has LinkedIn URL? -> Amplemarket LinkedIn lookup
       No LinkedIn?      -> Has domain? -> Amplemarket domain lookup
       No domain?        -> Mark as Unresolved
  -> If Amplemarket returned a count:
       -> Gemini: Is this a subsidiary? Estimate parent's global count
  -> Update HubSpot (employee count + enrichment source)
  -> Slack summary to #data-enrichment
```

## Enrichment Source Values

| Value | Meaning |
|-------|---------|
| `Amplemarket` | Direct Amplemarket data (not a subsidiary) |
| `Amplemarket (parent: {name})` | Subsidiary detected; Gemini estimated parent's global count |
| `Unresolved` | No data from Amplemarket (no LinkedIn URL or domain) |

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| HubSpot | hubspot | App Token | Fetch + update companies |
| Amplemarket | amplemarket | HTTP Header Auth | Company employee data |
| Google Gemini | Gemini | Google Palm API | Branch/subsidiary detection |
| Slack | Slack | Slack API | Summary notifications |

## HubSpot Properties

| Property | Type | Notes |
|----------|------|-------|
| `numberofemployees` | Standard | Employee count (number) |
| `number-employees-enrichment-source` | Custom | Enrichment source string |

## n8n Instance

- **Workflow ID**: `u9IcVLMFzBO6Idkw`
- **URL**: https://legalfly.app.n8n.cloud/workflow/u9IcVLMFzBO6Idkw
- **Error Workflow**: `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
- **Status**: Inactive (activate after testing)
