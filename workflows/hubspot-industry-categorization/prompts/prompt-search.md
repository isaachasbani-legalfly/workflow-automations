# Prompt: Search Fallback Path

**Node**: `Prepare Gemini Input - Search` (`prepare-gemini-search`)
**Triggered when**: No HubSpot description → LinkedIn failed/missing → no domain OR website scraping returned empty
**enrichmentSource**: `search`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{domain}}` | `Prepare Company Data` → `domain` (omitted from prompt if empty) |
| `{{searchContent}}` | Jina Reader response → `data`, preprocessed (max 2000 chars) |

## Search URL

```
GET https://r.jina.ai/https://duckduckgo.com/html/?q={{companyName}} company
```

Jina Reader fetches and parses the DuckDuckGo HTML search results page into clean text.

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

COMPANY DATA (from web search):
Name: {{companyName}}
Domain: {{domain}}               ← omitted entirely if domain is empty

Search Results:
{{searchContent}}

CLASSIFICATION RULES - Read carefully and apply in order:

1. TECHNOLOGY PRIORITY
   - If company CREATES software, SaaS platforms, apps, or tech products → Technology
   - Even if they serve a specific industry, if they BUILD the tech → Technology

2. LEGAL SERVICES
   - ANY legal work, legal consulting, compliance services → Legal Services
   - Legal is ALWAYS Legal Services, never Consulting

3. SPECIFIC SERVICE CATEGORIES
   - HR consulting, recruiting, staffing, payroll → HR and Payroll Services
   - Tax, bookkeeping, audit, CFO services → Accounting
   - Loans, deposits, payment processing → Banking
   - Investment advisory, wealth management → Financial Services

4. CONSTRUCTION vs MANUFACTURING
   - Building construction, infrastructure → Construction
   - Making products, goods, equipment → Manufacturing

5. RETAIL vs MANUFACTURING
   - If company sells products → Retail and Consumer Goods

6. PUBLIC SECTOR
   - ONLY government agencies → Public Sector
   - Non-profits → Others

7. MULTI-SERVICE COMPANIES
   - Multiple services → Consulting

8. OTHERS
   - Use ONLY when industry doesn't fit ANY of the 15 categories
   - Do NOT use Others as a fallback - always pick the best match

9. CONFLICT RESOLUTION
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
