# Prompt: Website Scraping Path

**Node**: `Prepare Gemini Input - Website` (`prepare-gemini-website`)
**Triggered when**: No HubSpot description → LinkedIn failed/missing → company has a domain → website returns meaningful content
**enrichmentSource**: `website`

## Input Variables

| Variable | Source |
|----------|--------|
| `{{companyName}}` | `Prepare Company Data` → `companyName` |
| `{{domain}}` | `Prepare Company Data` → `domain` |
| `{{websiteContent}}` | Jina Reader response → `data`, preprocessed (max 2000 chars) |

## Content Quality Gate

Before this node is reached, `Check Website Data Retrieved` now applies a smarter condition:

```
({ $json.data || '' })
  .replace(/cookie|consent|cookiebot|gdpr|datenschutz|impressum|privacy policy/gi, '')
  .replace(/\s+/g, ' ')
  .trim()
  .length > 300
```

If the scraped content is dominated by boilerplate (cookie banners, GDPR notices, consent dialogs), the stripped length falls below 300 characters → the IF node routes to **Jina Search** instead. Only pages with substantive content reach this node.

## Preprocessing Applied to Website Content

Before inserting into the prompt, the Jina Reader response (`$json.data`) is cleaned:
1. Strip markdown images: `![...](...)` → removed
2. Strip Wix asset links: `[...](https://static.wixstatic...)` → removed
3. Strip cookiebot links: `[...cookiebot...](...)` → removed
4. Strip cookie consent blocks: `This website uses cookies...` → removed
5. Strip consent selection blocks: `Consent Selection...` → removed
6. Strip individual boilerplate lines: cookiebot, cookie consent, cookie policy, GDPR, datenschutz, impressum → removed
7. Collapse 3+ consecutive blank lines → 2 blank lines
8. Trim whitespace
9. Truncate to **2000 characters**

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

COMPANY DATA (from website scraping):
Name: {{companyName}}
Domain: {{domain}}

Website Content:
{{websiteContent}}

HOW TO USE THIS DATA:
- Focus only on content that describes what the company does: services, products, mission, about sections
- Ignore completely: cookie consent notices, privacy policies, navigation menus, footer links, legal disclaimers
- If the content is dominated by boilerplate with no substantive company description, classify using the company name and domain as your only signals
- Company name and domain alone are sufficient to make a reasonable classification when content fails

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
5. Consulting — generalist advisory or professional services firms covering multiple domains
6. Public Sector — ONLY actual government agencies, not contractors
7. Others — ONLY when none of the 15 categories fit (agriculture, education, media, non-profits)

KEY DISTINCTIONS:
- Builds tech for any industry → Technology
- Any legal work → Legal Services, never Consulting
- Pharma, biotech, medical devices → Healthcare, not Retail and Consumer Goods
- Real estate developer (builds properties) → Construction
- Real estate advisory (manages/advises on properties) → Consulting
- Sells consumer products → Retail and Consumer Goods, beats Manufacturing
- Government contractor → their service category, not Public Sector
- Non-profit → Others

CRITICAL: Respond with ONLY the category name. No explanation, no punctuation, no extra text.

Examples:
- "We build SaaS software for law firms" → Technology
- "Law firm specializing in corporate law" → Legal Services
- "Biopharmaceutical company developing treatments for rare diseases" → Healthcare
- "HR consulting and recruiting services" → HR and Payroll Services
- "Bookkeeping and tax services for small businesses" → Accounting
- "Multi-service firm: accounting, HR, and legal advisory" → Consulting
- "Non-profit educational institution" → Others

Your response (category name only):
```
