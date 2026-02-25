# Gemini Prompt — Country Identification from Web Content (v2.2.3)

Used by the **Prepare Gemini Input** node (`v2-prep-gemini`). Gemini receives real web content scraped by Jina (website or DuckDuckGo search results) and must identify the country based ONLY on that content.

## Template

```
You are an expert at identifying the headquarters country of companies from web content.

COMPANY: {companyName}
DOMAIN: {domain}

WEB CONTENT:
{truncatedContent}

---

Based ONLY on the web content above, identify the headquarters country.

LOOK FOR (in order of reliability):
1. Physical address or registered office mentioned on the page
2. Phone number with international country code (e.g., +33 = France, +44 = UK, +32 = Belgium)
3. VAT/tax ID format (e.g., BE0xxx = Belgium, DE xxx = Germany, GB xxx = UK)
4. Company registration or legal entity type (GmbH = Germany/Austria/Switzerland, BV/NV = Netherlands/Belgium, Ltd/PLC = UK, SRL/SA = various, Inc/LLC = USA)
5. Explicit statement like "headquartered in [city], [country]" or "based in [country]"
6. Language and regional formatting clues (currency symbols, date formats)

RULES:
- Base your answer ONLY on the provided web content — do NOT use your own knowledge about company names or domains
- Do NOT confuse a company with another similarly named well-known company (e.g., "apollosuccess.io" is NOT "apollo.io", "wixsiteautomations.com" is NOT "wix.com")
- The evidence field MUST contain a direct quote from the web content, NOT your reasoning
- If the content is only error pages, login screens, or cookie banners with zero location clues: output Unknown

COUNTRY NAME RULES:
- Use full English country names
- NEVER use "England", "Scotland", "Wales", or "Northern Ireland" — always use "United Kingdom"
- NEVER use "Holland" — use "Netherlands"
- NEVER use "USA" or "US" — use "United States"
- NEVER use "UAE" — use "United Arab Emirates"

RESPONSE (JSON only, no markdown, no explanation):
{"country": "<full country name>", "evidence": "<exact quote from content>"}

If unknown:
{"country": "Unknown", "evidence": "No country indicators found in content"}
```

## Variables

| Variable | Source | Max length |
|----------|--------|------------|
| `companyName` | `$('Prepare Company Data').item.json.companyName` | -- |
| `domain` | `$('Prepare Company Data').item.json.domain` | -- |
| `truncatedContent` | Jina website scrape or DuckDuckGo search results | 6000 chars |

## Content Sources

1. **Website scrape** (primary): `$('Jina Website Scrape').item.json.data.content` — markdown of the company's homepage
2. **DuckDuckGo search** (fallback): `$('Jina Web Search').item.json.data.content` — scraped DuckDuckGo results page for `{companyName} headquarters country location`

## Content Cleaning

Before feeding to Gemini, the Prepare Gemini Input code node:
- Strips markdown images (`![...](...)`), image URLs (`.png`, `.jpg`, `.svg`, etc.)
- Removes cookie policy/notice/consent text
- **Website scrape**: Secondary quality check (rejects warnings + content < 200 chars)
- **DuckDuckGo search**: Strips navigation boilerplate before first numbered result, removes footer ("More results"), strips per-result action links (Block/Redo/Search domain), strips ad blocks
- Collapses triple+ newlines to double
- Truncates to 6000 chars

## Key Changes from v2.1

- **Anti-confusion rules**: Explicit instructions not to confuse similarly named companies, not to default to US for English content
- **Country name normalization**: "England" → "United Kingdom", "Holland" → "Netherlands", etc.
- **Evidence requirement**: Must quote actual text from content
- **Rebalanced from v2.2.2**: Removed "80% confidence threshold" and "when in doubt, output Unknown" — these were too aggressive and caused 60% unknown rate. Kept anti-confusion, country normalization, and evidence rules
