# HubSpot Industry Retroactive Enrichment

Manual workflow to backfill `industry__internal_` for companies that were created before the nightly categorization workflow was in place.

---

## When to use

Run this manually whenever you need to classify companies that are missing `industry__internal_`. Each execution processes **200 companies** — enough to stay well within n8n's 1-hour execution limit (~10 minutes per run).

**Total companies to backfill**: ~3,493
**Runs required**: ~18

---

## How it works

1. **Manual Trigger** — you click Execute in n8n
2. **Fetch 200 companies** — queries HubSpot for companies where `industry__internal_` is not set (fresh query each run, so already-classified companies never reappear)
3. **Split** — processes each company individually
4. **Classify** — same 4-path logic as the nightly workflow:
   - HubSpot description (≥100 chars) → Gemini
   - LinkedIn via Amplemarket → Gemini
   - Website scraping via Jina → Gemini
   - Web search via Jina → Gemini (last resort)
5. **Update HubSpot** — writes both `industry__internal_` and `industry_internal_enrichment_source`
6. **Slack summary** — sends results to channel `C0AG86U9927`

---

## Differences from nightly workflow

| | Nightly (v3.3) | Retroactive |
|--|--|--|
| Trigger | Cron (midnight) | Manual |
| Companies fetched | Created yesterday | Missing `industry__internal_` |
| Batch size | All from yesterday | 200 per run |
| Demo form skip | Yes | No (skipped for simplicity) |
| Slack channel | `D0ADELD95GR` | `C0AG86U9927` |
| Error isolation | No | `onError: continueRegularOutput` on all HTTP nodes |

---

## n8n instance

**Workflow ID**: `6EnyU8ofbAlq1bby`
**URL**: [https://legalfly.app.n8n.cloud/workflow/6EnyU8ofbAlq1bby](https://legalfly.app.n8n.cloud/workflow/6EnyU8ofbAlq1bby)
