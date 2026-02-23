# Gemini Grounded Inference Prompt

Used by node: **Prepare Gemini Grounded Input** (`v2-prep-grounded`)

## Purpose

Last-resort enrichment using Gemini with **Google Search grounding**. Gemini actively searches the web for company information, replacing the v1 Jina.ai scraping approach.

## Model Configuration

- **Model**: `gemini-2.5-flash`
- **Temperature**: `0.1`
- **Tools**: `[{"google_search": {}}]` (Google Search grounding enabled)
- **Retry**: 3 attempts, 5000ms between tries

## Prompt Template

```
You are an expert at identifying the headquarters country of companies.
Search the web for information about this company.

COMPANY: {companyName}
DOMAIN: {domain}

Search for this company's headquarters location, registered office, or primary
business address. Check their website, LinkedIn page, and business registries.

RESPONSE (JSON only, no markdown):
{"country": "<full country name in English>"}

RULES:
- Return the headquarters country only (not branch offices)
- Use full English country names (e.g., "United Kingdom", not "UK")
- If you cannot find reliable information: {"country": "Unknown"}
```

## Acceptance Criteria

- Valid country name (not "Unknown"): enrichment succeeds
- "Unknown" or parse failure: falls through to **Prepare Unknown** (country set to "Unknown" in HubSpot)
