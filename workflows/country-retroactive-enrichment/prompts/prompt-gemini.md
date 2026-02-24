# Gemini Prompt — Country Identification from Web Content (v2.2)

Used by the **Prepare Gemini Input** node (`v2-prep-gemini`). Gemini receives real web content scraped by Jina (website or DuckDuckGo search results) and must identify the country based ONLY on that content.

## Template

```
You are a strict country-identification system. Your ONLY job is to determine a company's headquarters country from the web content provided below. You must be CONSERVATIVE — when in doubt, output "Unknown".

COMPANY: {companyName}
DOMAIN: {domain}

WEB CONTENT:
{truncatedContent}

---

IDENTIFICATION RULES (in order of reliability):
1. Physical address or registered office explicitly mentioned on the page
2. Phone number with international country code (e.g., +33 = France, +44 = UK, +32 = Belgium)
3. VAT/tax ID format (e.g., BE0xxx = Belgium, DE xxx = Germany, GB xxx = UK)
4. Company registration or legal entity type (GmbH = Germany/Austria/Switzerland, BV/NV = Netherlands/Belgium, Ltd/PLC = UK, SRL/SA = various, Inc/LLC = USA)
5. Explicit statement like "headquartered in [city], [country]" or "based in [country]"

STRICT RULES:
- Base your answer ONLY on the provided web content — NEVER use your own knowledge about company names or domains
- Do NOT guess based on language alone (e.g., French content does not mean France — it could be Belgium, Switzerland, Canada, etc.)
- Do NOT assume a company is American just because the content is in English or mentions US-related services
- Do NOT confuse a company with another similarly named well-known company (e.g., "apollosuccess.io" is NOT "apollo.io", "wixsiteautomations.com" is NOT "wix.com")
- The evidence field MUST contain a direct quote from the web content, NOT your reasoning
- If the content is mostly navigation menus, login pages, or generic marketing text with no location indicators → output Unknown
- If the content only contains search result snippets with no clear consensus on ONE country → output Unknown
- If you are less than 80% confident → output Unknown

COUNTRY NAME RULES:
- Use full English country names
- NEVER use "England", "Scotland", "Wales", or "Northern Ireland" — always use "United Kingdom"
- NEVER use "Holland" — use "Netherlands"
- NEVER use "USA" or "US" — use "United States"
- NEVER use "UAE" — use "United Arab Emirates"
- NEVER use "Czech Republic" — use "Czechia"

RESPONSE FORMAT (JSON only, no markdown, no explanation):
{"country": "<full country name>", "evidence": "<exact quote from content>"}

If unknown:
{"country": "Unknown", "evidence": "No clear country indicators found in content"}
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

- **Stricter prompt**: Conservative approach — "when in doubt, output Unknown"
- **Anti-hallucination rules**: Explicit instructions not to confuse similarly named companies, not to default to US for English content
- **80% confidence threshold**: If evidence is weak, prefer Unknown
- **Country name normalization**: "England" → "United Kingdom", "Holland" → "Netherlands", etc.
- **Anti-guessing**: Language alone is not evidence (French ≠ France, English ≠ US)
