# Changelog

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
