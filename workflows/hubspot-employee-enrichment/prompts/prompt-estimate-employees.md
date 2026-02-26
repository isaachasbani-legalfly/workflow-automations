# Gemini Employee Estimation Prompt — Website Scrape Fallback

**Used by**: Prep Estimate Prompt (`emp2-prep-estimate`) + Gemini Estimate (`emp2-gemini-estimate`)
**Model**: gemini-2.5-flash
**Temperature**: 0.3

## Purpose

Estimates a company's employee count based on scraped website content. Used as a fallback when Amplemarket returns no data and the company has a domain to scrape.

## Prompt Template

```
You are analyzing a company website to estimate their employee count.

Company: {companyName}
Domain: {domain}

Website Content:
{scraped content, cleaned, max 3000 chars}

Estimate the number of employees. Consider:
- Team pages, office locations, about sections
- Industry norms and company size indicators
- Revenue/client indicators if mentioned
- Job listings or hiring pages

Return ONLY valid JSON (no markdown):
{"employeeCount": <number>, "confidence": "high"|"medium"|"low"}

If you cannot estimate, return: {"employeeCount": 5, "confidence": "low"}
```

## Expected Output

```json
{"employeeCount": 150, "confidence": "medium"}
```

## Content Cleaning

Before sending to Gemini, the scraped website content is cleaned:
- Markdown images stripped
- Wix static asset links removed
- Cookie consent/cookiebot blocks removed
- Excess blank lines collapsed
- Truncated to 3000 characters
- Content shorter than 50 characters is considered empty (triggers default path)

## Fallback Behavior

- Parse failure → default to 5 employees
- Empty/short content → skip Gemini, go directly to default (5 employees)
- All estimated values tagged with `enrichmentSource: "Estimated"`
