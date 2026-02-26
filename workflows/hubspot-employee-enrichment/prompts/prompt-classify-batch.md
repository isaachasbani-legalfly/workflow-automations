# Gemini Batch Classification Prompt — Subsidiary Detection

**Used by**: Prepare Gemini Batch (`emp2-prep-classify`)
**Model**: gemini-2.5-flash
**Temperature**: 0.2

## Purpose

Classifies ALL companies in a single batch call as either independent or subsidiary/branch/regional office. For subsidiaries, identifies the parent company and its primary domain.

## Prompt Template

```
You are an expert at identifying corporate subsidiaries, branches, and regional offices.

For each company below, determine if it is:
- An INDEPENDENT company (standalone business)
- A SUBSIDIARY/BRANCH/REGIONAL OFFICE of a larger parent company

Indicators of subsidiaries/branches:
- Country or city names in the company name (e.g., "KPMG Spain", "Deloitte UK", "Google Germany")
- Regional indicators (e.g., "EMEA", "Asia Pacific", "Latin America")
- "Branch", "Office", "Division", "Chapter" in the name
- Well-known subsidiary relationships (e.g., Instagram -> Meta, YouTube -> Alphabet)

COMPANIES TO CLASSIFY:
[0] "KPMG Spain" (domain: kpmg.es)
[1] "Acme Corp" (domain: acme.com)
...

Return ONLY a valid JSON array (no markdown, no explanation):
[{"index": 0, "isSubsidiary": false, "parentCompany": null, "parentDomain": null}]

Rules:
- For independent companies: isSubsidiary=false, parentCompany=null, parentDomain=null
- For subsidiaries: isSubsidiary=true, parentCompany=parent name, parentDomain=parent primary domain (e.g., "kpmg.com" not "kpmg.es")
- When unsure, default to independent (isSubsidiary=false)
```

## Expected Output

```json
[
  {"index": 0, "isSubsidiary": true, "parentCompany": "KPMG", "parentDomain": "kpmg.com"},
  {"index": 1, "isSubsidiary": false, "parentCompany": null, "parentDomain": null}
]
```

## Key Design Decisions

1. **Single batch call** — One API call classifies all companies (cheaper + faster than per-company)
2. **Low temperature (0.2)** — Deterministic, consistent classification
3. **Default to independent** — When uncertain, treat as standalone (less risky than false subsidiary detection)
4. **parentDomain = primary domain** — e.g., "kpmg.com" not "kpmg.es", so Amplemarket looks up the global parent
