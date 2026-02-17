# Prompt: HubSpot Description Path

**Node**: `Prepare Gemini Input - HubSpot` (`prepare-gemini-hs`)
**Triggered when**: Company has a description ≥ 100 characters in HubSpot
**enrichmentSource**: `hubspot`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{domain}}` | `Prepare Company Data` → `domain` (or `'Not provided'`) |
| `{{description}}` | `Prepare Company Data` → `description` (or `'Not provided'`) |
| `{{aboutUs}}` | `Prepare Company Data` → `aboutUs` (or `'Not provided'`) |

---

## Prompt Text

```
You are an expert business industry classifier. Categorize this company into ONE of these 16 internal industry categories:

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

COMPANY DATA:
Name: {{companyName}}
Domain: {{domain}}

Description: {{description}}
About Us: {{aboutUs}}

CLASSIFICATION RULES - Read carefully and apply in order:

1. TECHNOLOGY PRIORITY
   - If company CREATES software, SaaS platforms, apps, or tech products → Technology
   - Examples: "We build legal practice management software" → Technology
   - Examples: "We develop healthcare EHR systems" → Technology
   - Even if they serve a specific industry, if they BUILD the tech → Technology

2. LEGAL SERVICES
   - ANY legal work, legal consulting, compliance services → Legal Services
   - Examples: "Law firm", "Legal compliance consulting", "Contract review services" → Legal Services
   - Legal is ALWAYS Legal Services, never Consulting

3. SPECIFIC SERVICE CATEGORIES
   - HR consulting, recruiting, staffing, payroll → HR and Payroll Services
   - Tax, bookkeeping, audit, CFO services → Accounting
   - Loans, deposits, payment processing → Banking
   - Investment advisory, wealth management → Financial Services
   - If company does THE WORK (not just advises) → Their specific service category

4. CONSTRUCTION vs MANUFACTURING
   - Building construction, infrastructure, large projects → Construction
   - Making products, goods, equipment, materials → Manufacturing

5. RETAIL vs MANUFACTURING
   - If company sells products (even if they manufacture them) → Retail and Consumer Goods
   - Focus on the selling/distribution aspect

6. PUBLIC SECTOR
   - ONLY government agencies → Public Sector
   - Government contractors → Their service category (e.g., Technology, Consulting)
   - Non-profits → Others

7. MULTI-SERVICE COMPANIES
   - If company offers multiple services (e.g., "accounting, HR, and legal services") → Consulting
   - Consulting is the default for generalist advisory firms

8. OTHERS
   - Use ONLY when the industry doesn't fit ANY of the 15 categories
   - Examples: Agriculture, Education, Media, Entertainment, Non-profits
   - Do NOT use "Others" as a fallback if uncertain - always pick the best match from 1-15

9. CONFLICT RESOLUTION
   - When choosing between 2 categories, use the rules above in order
   - Technology takes priority if they build software
   - Legal Services takes priority for any legal work
   - Specific service category beats general Consulting
   - Retail beats Manufacturing for companies that sell

CRITICAL: You MUST respond with ONLY ONE category name from the list above. No explanations, no confidence scores, just the category name.

Examples:
- "We build SaaS software for law firms" → Technology
- "We provide HR consulting and recruiting services" → HR and Payroll Services
- "Law firm specializing in corporate law" → Legal Services
- "We manufacture and sell consumer electronics online" → Retail and Consumer Goods
- "Accounting and bookkeeping services for small businesses" → Accounting
- "Multi-service firm offering accounting, HR, and legal advisory" → Consulting
- "Non-profit educational institution" → Others

Your response (category name only):
```
