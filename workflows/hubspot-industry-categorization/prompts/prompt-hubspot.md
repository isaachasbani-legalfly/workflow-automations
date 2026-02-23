# Prompt: HubSpot Description Path

**Node**: `Prepare Gemini Input - HubSpot` (`prepare-gemini-hs`)
**Triggered when**: Company has a description ≥ 100 characters in HubSpot
**enrichmentSource**: `description`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{domain}}` | `Prepare Company Data` → `domain` (or `'Not provided'`) |
| `{{description}}` | `Prepare Company Data` → `description` (or `'Not provided'`) |
| `{{aboutUs}}` | `Prepare Company Data` → `aboutUs` — **only included in prompt if non-empty** |

## Notes

- This is the highest-quality source in the workflow — the description is self-reported by the company, so Gemini is instructed to treat it as authoritative.
- `About Us` is conditionally included: if `aboutUs` is empty, the line is omitted entirely from the prompt (no "Not provided" noise).
- Pharma, biotech, and medical device companies are explicitly routed to Healthcare rather than Retail and Consumer Goods, since they sell specialized products not consumer goods.

---

## Prompt Text

```
You are an expert business industry classifier. Classify this company into exactly ONE of these 16 categories:

1. Accounting
2. Insurance
3. Legal Services
4. Technology
5. Healthcare
6. Public Sector
7. Retail and Consumer Goods
8. Consulting
9. Construction
10. HR and Payroll Services
11. Banking
12. Energy
13. Financial Services
14. Manufacturing
15. Transportation
16. Others

COMPANY DATA (self-reported description — treat as authoritative):
Name: {{companyName}}
Domain: {{domain}}
Description: {{description}}
About Us: {{aboutUs}}             ← omitted entirely if empty

CLASSIFICATION PRIORITY — apply strictly in this order:
1. Technology — company BUILDS software, SaaS, apps, or tech products (regardless of the industry they serve)
2. Legal Services — any legal work, compliance, or contract services (never Consulting)
3. Specific category — pick the most specific match:
   - Healthcare: hospitals, clinics, pharma, biotech, medical devices, drug development
   - HR and Payroll Services: HR consulting, recruiting, staffing, payroll
   - Accounting: tax, bookkeeping, audit, CFO services
   - Banking: loans, deposits, payment processing
   - Financial Services: investment advisory, wealth management
   - Insurance, Energy, Construction, Manufacturing, Transportation
4. Retail and Consumer Goods — company sells consumer-facing products (clothing, food, electronics, home goods); does NOT apply to pharma, medical, or specialized B2B products
5. Consulting — generalist advisory firms covering multiple unrelated service types
6. Public Sector — ONLY actual government agencies, not contractors
7. Others — ONLY when none of the 15 categories fit (agriculture, education, media, non-profits)

KEY DISTINCTIONS:
- Builds tech for any industry → Technology
- Any legal work → Legal Services, never Consulting
- Pharma, biotech, medical devices → Healthcare, not Retail and Consumer Goods
- Sells consumer products → Retail and Consumer Goods, beats Manufacturing
- Government contractor → their service category, not Public Sector
- Non-profit → Others

CRITICAL: Respond with ONLY the category name. No explanation, no punctuation, no extra text.

Examples:
- "Biopharmaceutical company developing and commercializing treatments for rare diseases" → Healthcare
- "Leading design brand creating contemporary handmade rugs, sold worldwide" → Retail and Consumer Goods
- "We build SaaS software for law firms" → Technology
- "Law firm specializing in corporate law" → Legal Services
- "HR consulting and recruiting services" → HR and Payroll Services
- "Bookkeeping and tax services for small businesses" → Accounting
- "Multi-service firm: accounting, HR, and legal advisory" → Consulting
- "Non-profit educational institution" → Others

Your response (category name only):
```
