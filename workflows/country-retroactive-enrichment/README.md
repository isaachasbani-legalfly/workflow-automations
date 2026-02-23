# Country Retroactive Enrichment

Manual workflow to backfill the `country` property for HubSpot companies that have no country set.

---

## When to use

Run manually in n8n. Each execution processes **10 companies** (testing — increase to 200 for production runs).

---

## Classification logic

```
Has domain?
├── YES → Extract TLD → country-specific? (.de, .fr, .co.uk, .at, etc.)
│   ├── YES → Write country (source: Domain code) ✅
│   └── NO (generic: .com, .io, .net...) → Has LinkedIn URL?
│       ├── YES → Amplemarket → Got country?
│       │   ├── YES → Write country (source: Linkedin) ✅
│       │   └── NO → Has domain for scraping?
│       │       ├── YES → Jina + Gemini → Got country?
│       │       │   ├── YES → Write country (source: Website) ✅
│       │       │   └── NO → 🚩 Manual review
│       │       └── NO → 🚩 Manual review
│       └── NO → Has domain for scraping?
│           ├── YES → Jina + Gemini → Got country?
│           │   ├── YES → Write country (source: Website) ✅
│           │   └── NO → 🚩 Manual review
│           └── NO → 🚩 Manual review
└── NO domain → Has LinkedIn URL?
    ├── YES → Amplemarket → Got country?
    │   ├── YES → Write country (source: Linkedin) ✅
    │   └── NO → 🚩 Manual review
    └── NO → 🚩 Manual review
```

---

## HubSpot properties written

| Property | Values |
|----------|--------|
| `country` | Full country name (e.g. `Germany`, `United Kingdom`) |
| `countryenrichmentsource` | `Domain code` / `Linkedin` / `Website` / `Manual review` |

---

## Slack summary

Sent to channel `C0AG86U9927` at end of each run:
- ✅ Classified companies — name, country, source
- 🚩 Manual review — name + HubSpot link

---

## Gemini website prompt

When scraping, Gemini identifies HQ country using this signal priority:
1. Physical address / registered office on the page
2. Phone number country code
3. VAT / company registration number format
4. Language of content (weakest signal)

---

## n8n instance

**Workflow ID**: `SBCVJt431JqoysGl`
**URL**: [https://legalfly.app.n8n.cloud/workflow/SBCVJt431JqoysGl](https://legalfly.app.n8n.cloud/workflow/SBCVJt431JqoysGl)
