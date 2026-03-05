# Gemini Batch Classification Prompt — Subsidiary Detection

**Used by**: Prepare Gemini Batch (`emp2-prep-classify`)
**Model**: gemini-2.5-pro
**Temperature**: 0.2

## Purpose

Classifies ALL companies in a single batch call as either independent or subsidiary/branch. Much more conservative than v2.1 — only flags subsidiaries for unambiguous evidence. Target: 5-10% subsidiary rate.

## Prompt Template

```
You are an expert at identifying corporate subsidiaries and branches.

For each company below, determine if it is:
- INDEPENDENT (standalone business)
- SUBSIDIARY/BRANCH of a larger parent company

ONLY classify as SUBSIDIARY when you have UNAMBIGUOUS evidence from one of these categories:

1. FORTUNE GLOBAL 500 / BIG 4 with explicit geography:
   - "KPMG Spain", "Deloitte Netherlands", "PwC Austria" → subsidiary of the global firm
   - But "KPMG" alone (no geography) → INDEPENDENT (could be the parent itself)

2. EXPLICIT WORDS in the name: "Branch", "Office", "Division", "Filiale", "Niederlassung", "Sucursal"

3. FAMOUS ACQUISITIONS you are 100% certain about:
   - Instagram → Meta, YouTube → Alphabet, LinkedIn → Microsoft, WhatsApp → Meta
   - B&R → ABB, Numab → Johnson & Johnson
   - Do NOT guess acquisitions. If you are not 100% sure, classify as INDEPENDENT.

4. SUBDOMAINS of a known parent: e.g., deco.cscec.com → cscec.com

5. LINKEDIN URL contains "/branch/"

6. DUPLICATE ENTRIES: Two entries clearly being the same company with different TLDs (e.g., preh.com and preh.de)

ALWAYS INDEPENDENT — never classify these as subsidiaries:
- Government agencies, ministries, public authorities
- Universities, university hospitals, research institutes
- Non-profits, associations, foundations (e.V., Stiftung, Verband, Kuratorium)
- Airlines (e.g., Ita Airways, Vueling, Eurowings)
- Companies with geography in their BRAND NAME (e.g., Berlin Brands Group, Heidelberg Engineering, Swiss Life, Ita Airways)
- Companies with regional descriptors (Nordic, European, EMEA, Deutschland)
- Companies with country-code TLDs (.de, .es, .uk, .ch, .at) — this is NOT evidence of being a subsidiary

WHEN IN DOUBT → INDEPENDENT. A wrong "independent" label is harmless. A wrong "subsidiary" label causes incorrect data.

COMPANIES TO CLASSIFY:
[0] "KPMG Spain" (domain: kpmg.es)
[1] "Acme Corp" (domain: acme.com)
...

Return ONLY a valid JSON array (no markdown, no explanation):
[{"index": 0, "isSubsidiary": false, "parentCompany": null, "parentDomain": null}]

Rules:
- For independent companies: isSubsidiary=false, parentCompany=null, parentDomain=null
- For subsidiaries: isSubsidiary=true, parentCompany=parent name, parentDomain=parent primary domain (e.g., "kpmg.com" not "kpmg.es")
- When unsure, ALWAYS default to independent (isSubsidiary=false)
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
5. **Require unambiguous evidence** — Only 6 specific categories qualify. Everything else is independent.
6. **Exclude non-corporate entities** — Government, universities, hospitals, non-profits are always independent
7. **Airlines explicitly excluded** — Ita Airways, Vueling, Eurowings etc. are independent even though they may be owned by airline groups
8. **Geography-in-brand-name excluded** — Berlin Brands Group, Heidelberg Engineering, Swiss Life are brand names, not geographic subsidiaries

## Prompt History

- **v2.0**: Basic indicators (geography, regional names). Too aggressive — ~39% subsidiary rate.
- **v2.1**: Added "NOT indicators" and "STRONG EVIDENCE" requirement. Still too aggressive (~39%).
- **v3.0**: Complete rewrite. 6 specific evidence categories only. Explicit exclusion of airlines, geography-in-brand-name. Target: 5-10% subsidiary rate.
