# Changelog

## v2.1 (2026-02-24)

Replace Gemini internal knowledge with Jina web scraping + Gemini content analysis. Multiple iterations to optimize search, content quality, and Slack output.

### Added
- **Jina Website Scrape**: Scrapes company website via `r.jina.ai` to get real markdown content (10s timeout, no retries)
- **Check Website Data**: IF node with 3-condition quality gate (content notEmpty, no warning, length > 200)
- **Jina Web Search**: DuckDuckGo search via `r.jina.ai/https://duckduckgo.com/?q={companyName}` — fallback when scrape fails or no domain
- **Prepare Gemini Input**: Cleans/truncates content (3000 chars), strips DuckDuckGo boilerplate, builds evidence-based prompt
- **Gemini Inference**: Single Gemini call with NO tools — analyzes only the provided web content
- **Parse Gemini Response**: Extracts `{country, evidence}` from Gemini JSON response
- **Check Gemini Got Country**: Routes to result or unknown fallback

### Changed
- **Enrichment sources standardized**: Domain Code, Company Name, Amplemarket, Website, Web Search, Unresolved
- **Web search uses DuckDuckGo** instead of `s.jina.ai` — better search results, no rate limits
- **DuckDuckGo boilerplate stripping** in Prepare Gemini Input — removes navigation/footer before truncation
- **Website scrape quality gate** — rejects 404 pages, Wix errors, and content < 200 chars
- **Jina nodes have 10s timeout** and no retries to prevent hanging on dead domains
- **Slack summary**: Single message per execution with enriched/unknown sections and HubSpot View links
- **Format Summary uses `$('Restore Data').all()`** to collect all items across batches — sends one message
- **Aggregate node removed** — Format Summary now runs in `runOnceForAllItems` mode
- Gemini prompt now requires evidence (quoted text from page), not just a country name
- Slack summary version updated to v2.1
- Node count: 33 -> 31

### Removed
- **Gemini Blind Pass** (4 nodes): `v2-prep-blind`, `v2-gemini-blind`, `v2-parse-blind`, `v2-check-blind` — hallucinated on obscure companies
- **Gemini Grounded Pass** (4 nodes): `v2-prep-grounded`, `v2-gemini-grounded`, `v2-parse-grounded`, `v2-check-grounded` — black box, couldn't see what pages Gemini read
- **Aggregate All Results** node — replaced by `$('Restore Data').all()` pattern in Format Summary

### Why
- Gemini Blind classified "dewitlawoffice" as Netherlands (actually Belgium) — LLM internal knowledge is unreliable
- Gemini Grounded with google_search tool was a black box — no way to debug what pages were read
- `s.jina.ai` search API returned poor results — "kode legal" misclassified as US instead of UK
- DuckDuckGo via `r.jina.ai` provides real, visible search results with better accuracy
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
