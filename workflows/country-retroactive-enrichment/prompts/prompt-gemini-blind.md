# Gemini Blind Inference Prompt

Used by node: **Prepare Gemini Blind Input** (`v2-prep-blind`)

## Purpose

Infer company headquarters country from **name and domain only** (no web search, no scraping). Leverages Gemini's training data to recognize well-known companies and geographic signals.

## Model Configuration

- **Model**: `gemini-2.5-flash`
- **Temperature**: `0.1`
- **Tools**: None (pure knowledge inference)

## Prompt Template

```
You are an expert at identifying the headquarters country of companies.

COMPANY: {companyName}
DOMAIN: {domain}

Based ONLY on the company name and domain, identify the headquarters country.

SIGNALS:
1. Well-known company recognition (e.g., "Siemens" + siemens.com = Germany)
2. Geographic hints in domain name (e.g., "uk-consulting.com")
3. City/region references in name (e.g., "Berlin Digital GmbH")
4. Legal suffixes: GmbH=Germany/Austria/Switzerland, Ltd=UK, SRL=Italy/Romania,
   BV=Netherlands, SA=France/Spain, AB=Sweden, AS=Norway/Denmark,
   Pty=Australia/South Africa, Inc/LLC/Corp=United States

RESPONSE (JSON only, no markdown):
{"country": "<full country name>", "confidence": "HIGH|MEDIUM|LOW"}

HIGH = certain (recognized brand or unambiguous legal suffix)
MEDIUM = strong signals with some ambiguity
LOW = cannot determine confidently

If no idea: {"country": "Unknown", "confidence": "LOW"}
```

## Acceptance Criteria

- **HIGH** or **MEDIUM** confidence: country is accepted, enrichment stops
- **LOW** confidence: falls through to Gemini Grounded (Phase 4)
- Parse errors: treated as LOW confidence
