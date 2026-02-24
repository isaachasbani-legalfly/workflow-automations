# Changelog

## v2.1 (2026-02-24)

Replace Gemini internal knowledge with Jina web scraping + Gemini content analysis.

### Added
- **Jina Website Scrape**: Scrapes company website via `r.jina.ai` to get real markdown content
- **Check Website Data**: IF node to verify scrape returned content
- **Jina Web Search**: Fallback search via `s.jina.ai` for "{companyName} headquarters country" when scrape fails or no domain
- **Prepare Gemini Input**: Cleans/truncates content (3000 chars website, 2000 chars search), builds evidence-based prompt
- **Gemini Inference**: Single Gemini call with NO tools -- analyzes only the provided web content
- **Parse Gemini Response**: Extracts `{country, evidence}` from Gemini JSON response
- **Check Gemini Got Country**: Routes to result or unknown fallback

### Changed
- Enrichment source renamed from "Gemini blind" / "Gemini grounded" to "Jina+Gemini"
- Gemini prompt now requires evidence (quoted text from page), not just a country name
- Slack summary version updated to v2.1
- Node count: 33 -> 32 (removed 8, added 7)

### Removed
- **Gemini Blind Pass** (4 nodes): `v2-prep-blind`, `v2-gemini-blind`, `v2-parse-blind`, `v2-check-blind` -- hallucinated on obscure companies
- **Gemini Grounded Pass** (4 nodes): `v2-prep-grounded`, `v2-gemini-grounded`, `v2-parse-grounded`, `v2-check-grounded` -- black box, couldn't see what pages Gemini read

### Why
- Gemini Blind classified "dewitlawoffice" as Netherlands (actually Belgium) -- LLM internal knowledge is unreliable
- Gemini Grounded with google_search tool was a black box -- no way to debug what pages were read
- Jina provides real, visible web content that Gemini can only analyze (not hallucinate about)

## v2.0 (2026-02-23)

Major rewrite of the country enrichment workflow.

### Added
- **Pagination loop**: Fetches up to 200 companies per run (2 pages x 100) using cursor-based pagination
- **Gemini Blind Pass** (Phase 3): New zero-cost step that infers country from company name + domain using Gemini's training data, with structured JSON output and confidence scoring (HIGH/MEDIUM/LOW)
- **Gemini Google Search Grounding** (Phase 4): Replaces Jina.ai scraping with a single Gemini call using `google_search` tool for web-grounded inference
- **"Unknown" fallback**: Companies that fail all enrichment phases get `country="Unknown"` written to HubSpot, preventing infinite retry loops
- **Source distribution stats**: Slack summary now shows breakdown by enrichment source (TLD, Company name, Amplemarket, Gemini blind, Gemini grounded)
- **Snapshot Data pattern**: Set node before HubSpot update preserves result data for post-update aggregation

### Changed
- Enrichment cascade simplified from 8 steps to 5 phases
- Amplemarket consolidated from 3 possible calls (LinkedIn, Domain, Website-LinkedIn) to single domain call
- Gemini temperature lowered from 0.3 to 0.1 for more deterministic output
- HubSpot search limit increased from 10 to 100 per page

### Removed
- **Jina.ai** scraping completely removed (was 20 RPM bottleneck)
- **LinkedIn-based Amplemarket** calls removed (3 nodes)
- **Website LinkedIn extraction** removed (2 nodes)
- **Manual Review** path replaced with "Unknown" fallback

## v1.0 (2026-02-14)

Initial implementation with 8-step enrichment cascade.
- TLD extraction, name scan, LinkedIn Amplemarket, domain Amplemarket, Jina scraping, LinkedIn extraction from website, website LinkedIn Amplemarket, Gemini inference
- No pagination (limit: 10)
- No "Unknown" fallback
