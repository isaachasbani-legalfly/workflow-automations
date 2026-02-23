# Prompt: LinkedIn Path

**Node**: `Prepare Gemini Input - LinkedIn` (`prepare-gemini-linkedin`)
**Triggered when**: No HubSpot description → has LinkedIn URL → Amplemarket returns data
**enrichmentSource**: `linkedin`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{industry}}` | Amplemarket response → `industry` (or `'Not provided'`) |
| `{{keywords}}` | Amplemarket response → `keywords` joined by `, ` (or `'Not provided'`) |
| `{{overview}}` | Amplemarket response → `overview` (or `'Not provided'`) |

## Notes

- Keywords and LinkedIn Industry are the primary signals — they reflect actual service areas.
- Overview is often a single marketing line and should be treated as supporting context only.
- LinkedIn's industry taxonomy differs from ours — explicit mapping guidance is included in the prompt.
- Pharma, biotech, and medical device companies are explicitly routed to Healthcare rather than Retail and Consumer Goods.

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

COMPANY DATA (from LinkedIn):
Name: {{companyName}}
LinkedIn Industry: {{industry}}
Keywords: {{keywords}}

Overview:
{{overview}}

HOW TO USE THIS DATA:
- Keywords and LinkedIn Industry are your primary signals — they reflect actual service areas
- Overview is often a single marketing line — use it as supporting context only, not the primary basis for classification
- LinkedIn's industry taxonomy differs from ours: use the mapping guidance below when needed

LINKEDIN INDUSTRY MAPPING GUIDANCE:
- "Information Technology and Services", "Software", "Internet" → Technology (if they build products) or Consulting
- "Financial Services", "Investment Management", "Venture Capital" → Financial Services or Banking
- "Staffing and Recruiting", "Human Resources" → HR and Payroll Services
- "Accounting" → Accounting
- "Legal Services", "Law Practice" → Legal Services
- "Real Estate" → Consulting (if advisory/services) or Construction (if they build)
- "Insurance" → Insurance
- "Hospital & Health Care", "Medical Devices", "Pharmaceuticals", "Biotechnology" → Healthcare
- "Government Administration" → Public Sector
- When LinkedIn's industry doesn't map cleanly, rely on Keywords to determine the right category

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
5. Consulting — multi-service advisory or professional services firms covering multiple domains
6. Public Sector — ONLY actual government agencies, not contractors
7. Others — ONLY when none of the 15 categories fit (agriculture, education, media, non-profits)

KEY DISTINCTIONS:
- Builds tech for any industry → Technology
- Any legal work → Legal Services, never Consulting
- Pharma, biotech, medical devices → Healthcare, not Retail and Consumer Goods
- Real estate advisory or property services → Consulting
- Sells consumer products → Retail and Consumer Goods, beats Manufacturing
- Government contractor → their service category, not Public Sector
- Non-profit → Others

CRITICAL: Respond with ONLY the category name. No explanation, no punctuation, no extra text.

Examples:
- "Commercial real estate services: valuation, asset management, property advisory" → Consulting
- "Biopharmaceutical company developing and commercializing treatments for rare diseases" → Healthcare
- "We build SaaS software for law firms" → Technology
- "Law firm specializing in corporate law" → Legal Services
- "HR consulting and recruiting services" → HR and Payroll Services
- "Bookkeeping and tax services for small businesses" → Accounting
- "Investment advisory and wealth management" → Financial Services
- "Non-profit educational institution" → Others

Your response (category name only):
```
