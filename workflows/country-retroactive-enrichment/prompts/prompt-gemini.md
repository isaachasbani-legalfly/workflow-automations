# Gemini Prompt — Country Identification from Web Content (v2.1)

Used by the **Prepare Gemini Input** node (`v2-prep-gemini`). Gemini receives real web content scraped by Jina (website or search results) and must identify the country based ONLY on that content.

## Template

```
You are an expert at identifying the headquarters country of companies.
Below is web content scraped from this company's online presence.

COMPANY: {companyName}
DOMAIN: {domain}

WEB CONTENT:
{truncatedContent}

Based ONLY on the web content above, identify the headquarters country.

LOOK FOR (in order of reliability):
1. Physical address or registered office mentioned on the page
2. Phone number with international country code
3. VAT number, company registration number, or tax ID format
4. Legal entity type (e.g., GmbH, Ltd, BV, NV, SRL, SA, AB, Pty, Inc, LLC)
5. Language and regional formatting clues (currency symbols, date formats)

RESPONSE (JSON only, no markdown):
{"country": "<full country name>", "evidence": "<quote the specific text from the content that confirmed this>"}

RULES:
- Base your answer ONLY on the provided web content — do NOT use your own knowledge
- Use full English country names (e.g., "United Kingdom", not "UK")
- The evidence field MUST quote actual text from the page, not your reasoning
- If the content doesn't contain clear country indicators: {"country": "Unknown", "evidence": "No country indicators found in content"}
```

## Variables

| Variable | Source | Max length |
|----------|--------|------------|
| `companyName` | `$('Prepare Company Data').item.json.companyName` | -- |
| `domain` | `$('Prepare Company Data').item.json.domain` | -- |
| `truncatedContent` | Jina website scrape or search results | 3000 chars (website) / 2000 chars (search) |

## Content Sources

1. **Website scrape** (primary): `$('Jina Website Scrape').item.json.data.content` -- markdown of the company's homepage
2. **Web search** (fallback): `$('Jina Web Search').item.json.data[]` -- top 5 search results for "{companyName} headquarters country"

## Content Cleaning

Before feeding to Gemini, the Prepare Gemini Input code node:
- Strips markdown images (`![...](...)`)
- Removes image URLs (`.png`, `.jpg`, `.svg`, etc.)
- Removes cookie policy/notice/consent text
- Collapses triple+ newlines to double
- Truncates to max chars limit

## Key Differences from v2.0

- **v2.0 blind pass**: Used Gemini's internal knowledge (no web content) -- hallucinated on obscure companies
- **v2.0 grounded pass**: Used `google_search` tool -- black box, couldn't see what pages were read
- **v2.1**: Jina scrapes real content, feeds it to Gemini with NO tools -- full transparency and evidence-based
