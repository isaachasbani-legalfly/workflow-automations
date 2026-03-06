# Gemini Batch Classification Prompt — Subsidiary Detection

**Used by**: Prepare Gemini Batch (`emp2-prep-classify`)
**Model**: gemini-2.5-pro
**Temperature**: 0.2

## Purpose

Classifies ALL companies in a single batch call as either independent or subsidiary/branch, with a confidence level. Two-tier confidence system: HIGH (auto-write) and MEDIUM (auto-write + flagged for review).

## Prompt Template

```
You are an expert at identifying corporate subsidiaries and group companies.

For each company below, determine if it is:
- INDEPENDENT (standalone business)
- SUBSIDIARY/BRANCH of a larger parent company or group

Classify as SUBSIDIARY only when you have strong evidence. Return a confidence level:

HIGH CONFIDENCE — unambiguous, well-known relationships:

1. WELL-KNOWN GLOBAL FIRM + EXPLICIT GEOGRAPHY/REGION in the name:
   - "KPMG Spain" -> KPMG, "Deloitte Netherlands" -> Deloitte, "PwC Austria" -> PwC
   - The parent is the same brand without the geography
   - Must be a universally recognized global brand (Big 4, MBB, global law firms, etc.)
   - But "KPMG" alone (no geography) -> INDEPENDENT (could be the parent itself)

2. SAME BRAND + OPERATIONAL SUFFIX in the name:
   - "Weber-Stephen Products EMEA" -> Weber-Stephen Products
   - "Jones Bros Civil Engineering UK" -> Jones Bros Civil Engineering
   - The base name is identical, only a region/division word is added (EMEA, UK, Europe, Deutschland, Nordic, etc.)

3. UNIVERSALLY KNOWN OWNERSHIP — relationships that are common public knowledge:
   - Tech: Instagram -> Meta, YouTube -> Alphabet, WhatsApp -> Meta, LinkedIn -> Microsoft, GitHub -> Microsoft
   - Luxury: Louis Vuitton -> LVMH, Fendi -> LVMH, Dior -> LVMH, Gucci -> Kering, Balenciaga -> Kering
   - Industrial: Porsche -> Volkswagen Group
   - If you have to think twice about whether it's universally known, it's NOT universally known.

MEDIUM CONFIDENCE — likely but not universally famous:

4. LIKELY DIVISION/BRAND of a parent company:
   - The name strongly suggests it's a division (e.g., "Keepmoat Regeneration" -> Keepmoat)
   - The parent relationship is plausible but not common knowledge
   - You are fairly confident but it's not a household-name relationship

ALWAYS INDEPENDENT — never classify these as subsidiaries:
- Government agencies, ministries, public authorities
- Universities, university hospitals, research institutes
- Non-profits, associations, foundations (e.V., Stiftung, Verband, Kuratorium)
- Airlines (e.g., Ita Airways, Vueling, Eurowings) — even if owned by a group
- Companies with geography in their BRAND NAME (e.g., Berlin Brands Group, Heidelberg Engineering, Swiss Life, Ita Airways)
- Consumer email/cloud domains (icloud.com, me.com, gmail.com, outlook.com) — these are NOT company domains
- When the "parent" would be THE SAME ENTITY (e.g., "Kotak Mahindra" is NOT a subsidiary of "Kotak Mahindra")
- When in doubt -> INDEPENDENT. A wrong "independent" is harmless. A wrong "subsidiary" causes incorrect data.

COMPANIES TO CLASSIFY:
[0] "KPMG Spain" (domain: kpmg.es)
[1] "Acme Corp" (domain: acme.com)
...

NOTE: Only company name and domain are provided. Do NOT use domain patterns (TLDs, subdomains) as evidence of subsidiary relationships. Classification is based purely on company name and your knowledge of well-known corporate structures.

Return ONLY a valid JSON array (no markdown, no explanation):
[{"index": 0, "isSubsidiary": true, "confidence": "high", "parentCompany": "KPMG", "parentDomain": "kpmg.com"}]

Rules:
- For independent companies: isSubsidiary=false, confidence=null, parentCompany=null, parentDomain=null
- For high-confidence subsidiaries: isSubsidiary=true, confidence="high", parentCompany=parent name, parentDomain=parent primary domain
- For medium-confidence subsidiaries: isSubsidiary=true, confidence="medium", parentCompany=parent name, parentDomain=parent primary domain
- When unsure, ALWAYS default to independent (isSubsidiary=false)
```

## Expected Output

```json
[
  {"index": 0, "isSubsidiary": true, "confidence": "high", "parentCompany": "KPMG", "parentDomain": "kpmg.com"},
  {"index": 1, "isSubsidiary": false, "confidence": null, "parentCompany": null, "parentDomain": null},
  {"index": 2, "isSubsidiary": true, "confidence": "medium", "parentCompany": "Keepmoat", "parentDomain": "keepmoat.com"}
]
```

## Confidence Tiers

| Tier | Auto-write | Slack flag | Example |
|------|-----------|------------|---------|
| HIGH | Yes | No | KPMG Spain -> KPMG, Instagram -> Meta |
| MEDIUM | Yes | Yes (flagged for review) | Keepmoat Regeneration -> Keepmoat |

## Key Design Decisions

1. **Single batch call** — One API call classifies all companies
2. **Low temperature (0.2)** — Deterministic, consistent classification
3. **Default to independent** — When uncertain, treat as standalone
4. **No domain-based evidence** — Domain patterns (TLDs, subdomains) are NOT used. Only company name and Gemini's knowledge of corporate structures.
5. **Two confidence tiers** — Both get written to HubSpot, but MEDIUM is flagged in Slack for manual review
6. **Self-referencing blocked** — A company cannot be a subsidiary of itself
7. **Airlines excluded** — Even if owned by a group (Ita Airways owned by Lufthansa Group), they are independent for our purposes
8. **Geography-in-brand-name excluded** — Berlin Brands Group, Swiss Life, etc.
9. **No LinkedIn URLs** — HubSpot LinkedIn data is often incorrect

## Prompt History

- **v2.0**: Basic indicators (geography, regional names). Too aggressive (~39% subsidiary rate).
- **v2.1**: Added "NOT indicators" and "STRONG EVIDENCE" requirement. Still too aggressive (~39%).
- **v3.0**: Complete rewrite. 5 specific evidence categories (name + domain only, no LinkedIn). Target: 5-10%.
- **v4.0**: Removed domain-based rules (subdomains, TLDs). Added confidence tiers (HIGH/MEDIUM). Famous acquisitions/ownership moved to HIGH. Likely divisions moved to MEDIUM.
