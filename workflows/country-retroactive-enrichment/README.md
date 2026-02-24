# Country Retroactive Enrichment

Manual workflow to backfill the `country` property for HubSpot companies that have no country set.

---

## When to use

Run manually in n8n. Each execution processes up to **200 companies** (2 pages x 100, configurable via `maxPages` in Initialize State).

---

## Enrichment Cascade (v2.1)

```
Phase 1 — Zero-cost (instant):
  Has domain? → Extract TLD (.de → Germany, .co.uk → UK, etc.)
  TLD failed? → Scan company name for country names / demonyms

Phase 2 — Amplemarket (single domain call):
  Has domain? → GET /companies/find?domain= → headquarters_country

Phase 3 — Jina + Gemini (replaces blind + grounded):
  Has domain? → Jina scrape website → got content?
  No content or no domain → Jina web search "{companyName} headquarters country"
  Feed real content to Gemini → extract country + evidence

Phase 4 — Fallback:
  All methods failed → country = "Unknown"
```

---

## HubSpot properties written

| Property | Values |
|----------|--------|
| `country` | Full country name (e.g. `Germany`, `United Kingdom`) or `Unknown` |
| `countryenrichmentsource` | `TLD` / `Company name` / `Amplemarket` / `Jina+Gemini` / `Unresolved` |

---

## Slack summary

Sent to channel `C0AG86U9927` at end of each run:
- Total companies processed
- Source distribution stats (TLD, Company name, Amplemarket, Jina+Gemini, Unresolved)
- List of enriched companies with country and source

---

## Credentials required

| Service | Credential | Type |
|---------|-----------|------|
| HubSpot | `hubspot` | hubspotAppToken |
| Google Gemini | `Gemini` | googlePalmApi |
| Amplemarket | `amplemarket` | httpHeaderAuth |
| Jina | *(inline Bearer token)* | sendHeaders |
| Slack | `Slack` | slackApi |

---

## Version history

| Version | Date | Key changes |
|---------|------|-------------|
| v2.1 | 2026-02-24 | Replace Gemini blind + grounded with Jina scrape/search + Gemini content analysis. Eliminates LLM hallucination and adds full transparency |
| v2.0 | 2026-02-23 | Pagination loop, Gemini blind pass, Gemini Google Search grounding, "Unknown" fallback, source distribution stats |
| v1.0 | 2026-02-14 | Initial 8-step cascade (TLD, name, LinkedIn Amplemarket, domain Amplemarket, Jina scraping, website LinkedIn, Gemini inference) |

---

## n8n instance

| Version | Workflow ID | URL |
|---------|------------|-----|
| **v2.1** (current) | `h4Dwz3Z2bhksWYly` | [https://legalfly.app.n8n.cloud/workflow/h4Dwz3Z2bhksWYly](https://legalfly.app.n8n.cloud/workflow/h4Dwz3Z2bhksWYly) |
| v1.0 (legacy) | `SBCVJt431JqoysGl` | [https://legalfly.app.n8n.cloud/workflow/SBCVJt431JqoysGl](https://legalfly.app.n8n.cloud/workflow/SBCVJt431JqoysGl) |
