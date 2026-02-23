# Prompt: Search Fallback Path

**Node**: `Prepare Gemini Input - Search` (`prepare-gemini-search`)
**Triggered when**: No HubSpot description → LinkedIn failed/missing → no domain OR website returned boilerplate
**enrichmentSource**: `search`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{domain}}` | `Prepare Company Data` → `domain` (omitted from prompt if empty) |
| `{{searchContent}}` | Jina Reader response → `data`, preprocessed (max 2000 chars) |

## Search URL

```
GET https://r.jina.ai/https://duckduckgo.com/html/?q={{cleanedName}} company
```

Jina Reader fetches and parses the DuckDuckGo HTML search results page into clean text.

**Name cleaning**: Before building the query, if `companyName` looks like a domain (e.g. `steewart.com`, `promos-consult.de`), the TLD is stripped and hyphens/dots are replaced with spaces:
- `steewart.com` → `steewart company`
- `promos-consult.de` → `promos consult company`
- `BUWOG` → `BUWOG company` (unchanged — not a domain)

## Preprocessing Applied to Search Content

Before inserting into the prompt, the Jina response (`$json.data`) is cleaned:
1. Strip markdown images: `![...](...)` → removed
2. Strip links, keep link text: `[text](url)` → `text`
3. Collapse 3+ consecutive blank lines → 2 blank lines
4. Trim whitespace
5. Truncate to **2000 characters**

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

COMPANY DATA (from web search — last resort source):
Name: {{companyName}}
Domain: {{domain}}               ← omitted entirely if domain is empty

Search Results:
{{searchContent}}

HOW TO USE THIS DATA:
- Search results may include multiple companies with similar names — prioritise results that match the company name or domain exactly
- When the company name is a domain (e.g. "silvercord.eu"), search results may mix different companies; use the most relevant match
- Source quality hierarchy: company's own website > LinkedIn profile > Wikipedia > business directories > ads or unrelated results
- When multiple results conflict, pick the most specific category supported by the most authoritative source
- When results are only ads, sponsored links, or completely unrelated to the company, classify using the domain name alone as your signal
- If the domain gives no meaningful signal either, use Others

CLASSIFICATION PRIORITY — apply strictly in this order:
1. Technology — company BUILDS software, SaaS, apps, or tech products (regardless of the industry they serve)
2. Legal Services — any legal work, compliance, or contract services (never Consulting)
3. Specific category — pick the most specific match:
   - Healthcare: hospitals, clinics, pharma, biotech, medical devices, drug development
   - HR and Payroll Services: HR consulting, recruiting, staffing, payroll
   - Accounting: tax, bookkeeping, audit, CFO services
   - Banking: loans, deposits, payment processing
   - Financial Services: investment advisory, wealth management, title insurance
   - Insurance, Energy, Construction, Manufacturing, Transportation
4. Retail and Consumer Goods — company sells consumer-facing products (clothing, food, electronics, home goods); does NOT apply to pharma, medical, or specialized B2B products
5. Consulting — generalist advisory or professional services firms covering multiple domains
6. Public Sector — ONLY actual government agencies, not contractors
7. Others — when search results are empty/ads only AND domain gives no signal; or when no category fits

KEY DISTINCTIONS:
- Builds tech for any industry → Technology (e.g. SAP software for real estate → Technology)
- Any legal work → Legal Services, never Consulting
- Pharma, biotech, medical devices → Healthcare, not Retail and Consumer Goods
- Title insurance, real estate services → Financial Services or Insurance
- Sells consumer products → Retail and Consumer Goods, beats Manufacturing
- Government contractor → their service category, not Public Sector
- Non-profit → Others

CRITICAL: Respond with ONLY the category name. No explanation, no punctuation, no extra text.

Examples:
- Search returns results for a title insurance company → Insurance
- Search shows "develops SAP-based software for real estate digitalisation" → Technology
- Search returns only ads with no company information, domain gives no signal → Others
- Search returns conflicting results, clearest source shows recruitment/staffing → HR and Payroll Services
- "Law firm specializing in corporate law" → Legal Services
- "Biopharmaceutical company developing treatments for rare diseases" → Healthcare
- "Bookkeeping and tax services for small businesses" → Accounting
- "Non-profit educational institution" → Others

Your response (category name only):
```
