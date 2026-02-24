# Changelog

All notable changes to the Country Retroactive Enrichment workflow will be documented in this file.

---

## [2.2.1] - 2026-02-24

### Fixed — IF node operator + LinkedIn extraction from website scrape

**Fix 1 — IF node operator name**: The two new IF nodes (`v2-check-linkedin`, `v2-check-amp-linkedin`) used `"isNotEmpty"` operator, but n8n v2.2 IF nodes require `"notEmpty"`. Updated both to `typeVersion: 2.2` with correct operator. This was causing all items to route to FALSE (bypassing Amplemarket LinkedIn lookup entirely).

**Fix 2 — LinkedIn extraction from website scrape**: The Extract LinkedIn URL node previously only parsed DuckDuckGo search results. Now it also parses Jina Website Scrape content (company websites often have LinkedIn links in their footer). Check Website Data TRUE now routes to Extract LinkedIn URL instead of directly to Prepare Gemini Input.

**Connection change**: Check Website Data TRUE → Extract LinkedIn URL (was → Prepare Gemini Input). Both website scrape and web search paths now merge into the same LinkedIn extraction chain before falling through to Gemini.

**Workflow ID**: `h4Dwz3Z2bhksWYly`

---

## [2.2.0] - 2026-02-24

### Added — Better search query + LinkedIn URL extraction for Amplemarket

**Problem**: 49/100 companies still classified as "Unknown" after v2.1 fixes. Root causes:
1. DuckDuckGo returns irrelevant "Related Searches" sidebar content for generic company names
2. No mechanism to leverage LinkedIn company URLs that appear in search results

**Fix 1 — Better search query**:
- Added `headquarters country location` to DuckDuckGo search query
- Before: `?q={companyName}` → After: `?q={companyName} headquarters country location`
- More targeted results with location-relevant content

**Fix 2 — LinkedIn URL → Amplemarket lookup** (5 new nodes):
- **Extract LinkedIn URL** (`v2-extract-linkedin`): Parses `linkedin.com/company/` URLs from DuckDuckGo search results
- **Check Has LinkedIn URL** (`v2-check-linkedin`): Routes items with LinkedIn URLs to Amplemarket
- **Amplemarket LinkedIn Lookup** (`v2-amp-linkedin`): `GET /companies/find?linkedin_url=` — same API, different lookup key
- **Parse LinkedIn Amplemarket Country** (`v2-parse-amp-linkedin`): Extracts primary location country from response
- **Check LinkedIn Amplemarket Got Country** (`v2-check-amp-linkedin`): Routes to Prepare Result (success) or Prepare Gemini Input (fallback)

**Flow change**: Jina Web Search no longer connects directly to Prepare Gemini Input. Instead:
```
Jina Web Search → Extract LinkedIn URL → Has LinkedIn?
  YES → Amplemarket LinkedIn → Parse Country → Got Country?
    YES → Prepare Result (done!)
    NO  → Prepare Gemini Input (continue)
  NO  → Prepare Gemini Input (continue)
```

**Compatibility**: The existing Prepare Result node already handles `countryFromAmplemarket` → source "Amplemarket". No changes needed downstream.

**Workflow ID**: `h4Dwz3Z2bhksWYly`
**Total nodes**: 36 (was 31)

---

## [2.1.3] - 2026-02-24

### Fixed — Single Slack message via `$getWorkflowStaticData`

**Root cause**: When items take different paths through IF nodes in n8n, they arrive at downstream convergence nodes as separate batches. Each batch triggers the downstream node independently. Neither `$('NodeName').all()` nor the Aggregate node reliably waits for all batches — the Aggregate node also processes each batch independently.

**Fix**: Removed the Aggregate node. Rewrote Format Summary to use `$getWorkflowStaticData('node')` to accumulate results across batches:
- Stores results in persistent static data, keyed by `$execution.id` to reset between runs
- Returns `[]` (empty — Slack won't trigger) until `staticData._results.length >= totalExpected`
- Only the final batch produces the Slack message
- Uses `_sent` flag to prevent duplicate messages

**Workflow ID**: `h4Dwz3Z2bhksWYly`

---

## [2.1.2] - 2026-02-24

### Fixed — HubSpot `country` property not being written (countryRegion)

**Root cause**: `country` is a **standard** HubSpot company property. Writing it via `customPropertiesUi` in the n8n HubSpot node silently fails — the `countryenrichmentsource` (a custom property) wrote fine, but `country` came back as `null` from HubSpot. This caused companies to reappear on every run because the `NOT_HAS_PROPERTY` filter kept matching them.

**Evidence**: Snapshot Data showed correct values (`"United States"`, `"United Kingdom"`), but the HubSpot API response returned `country: null`.

**Fix**: Moved `country` to the built-in `updateFields.countryRegion` field (which maps to the standard HubSpot `country` property). Kept only `countryenrichmentsource` in `customPropertiesUi`.

**Workflow ID**: `h4Dwz3Z2bhksWYly`

---

## [2.1.1] - 2026-02-24

### Fixed — Format Summary returning empty (no Slack message sent)

**Root cause**: `$('Restore Data').all()` only returns items that have been processed so far when each batch arrives. The first batch (3 TLD-resolved items) arrived, saw 3/10 total, returned `[]`. All subsequent batches also had fewer than 10 items visible.

**First fix attempt**: Added Aggregate node — didn't solve it because the Aggregate node also processes each batch independently in n8n.

**Fix**: See v2.1.3 for the final `$getWorkflowStaticData` solution.

**Workflow ID**: `h4Dwz3Z2bhksWYly`

---

## [2.1.0] - 2026-02-24

### Changed — Replace Gemini blind + grounded with Jina + Gemini content analysis

**Added** (6 nodes):
- **Jina Website Scrape**: Scrapes company website via `r.jina.ai` to get real markdown content (10s timeout, no retries)
- **Check Website Data**: 3-condition quality gate (content notEmpty, no warning, length > 200)
- **Jina Web Search**: DuckDuckGo search via `r.jina.ai/https://duckduckgo.com/?q={companyName}` — fallback when scrape fails or no domain
- **Prepare Gemini Input**: Cleans/truncates content (3000 chars), strips DuckDuckGo boilerplate, builds evidence-based prompt
- **Gemini Inference**: Single Gemini call with NO tools — analyzes only the provided web content
- **Parse Gemini Response**: Extracts `{country, evidence}` from Gemini JSON response
- **Check Gemini Got Country**: Routes to result or unknown fallback

**Removed** (8 nodes):
- Gemini Blind Pass: `v2-prep-blind`, `v2-gemini-blind`, `v2-parse-blind`, `v2-check-blind` — hallucinated on obscure companies
- Gemini Grounded Pass: `v2-prep-grounded`, `v2-gemini-grounded`, `v2-parse-grounded`, `v2-check-grounded` — black box, couldn't see what pages Gemini read

**Changed**:
- Enrichment sources standardized: Domain Code, Company Name, Amplemarket, Website, Web Search, Unresolved
- Web search uses DuckDuckGo via `r.jina.ai` instead of `s.jina.ai` — better accuracy
- Jina nodes have 10s timeout and no retries to prevent hanging on dead domains
- Gemini prompt requires evidence (quoted text from page), not just a country name

**Why**:
- Gemini Blind classified "dewitlawoffice" as Netherlands (actually Belgium) — LLM internal knowledge is unreliable
- Gemini Grounded with `google_search` tool was a black box — no way to debug what pages were read
- `s.jina.ai` search API returned poor results — "kode legal" misclassified as US instead of UK

**Workflow ID**: `h4Dwz3Z2bhksWYly`

---

## [2.0.0] - 2026-02-23

### Changed — Major rewrite of the country enrichment workflow

**Added**:
- Pagination loop: Fetches up to 200 companies per run using cursor-based pagination
- Gemini Blind Pass (Phase 3): Infers country from company name + domain using Gemini's training data
- Gemini Google Search Grounding (Phase 4): Single Gemini call using `google_search` tool
- "Unknown" fallback: Companies that fail all phases get `country="Unknown"` written to HubSpot
- Snapshot Data pattern: Set node before HubSpot update preserves result data

**Changed**:
- Enrichment cascade simplified from 8 steps to 5 phases
- Amplemarket consolidated to single domain call
- Gemini temperature lowered from 0.3 to 0.1

**Removed**:
- Jina.ai scraping (was 20 RPM bottleneck)
- LinkedIn-based Amplemarket calls
- Manual Review path (replaced with "Unknown" fallback)

---

## [1.0.0] - 2026-02-14

### Initial Release

- 8-step enrichment cascade: TLD extraction, name scan, LinkedIn Amplemarket, domain Amplemarket, Jina scraping, LinkedIn extraction from website, website LinkedIn Amplemarket, Gemini inference
- No pagination (limit: 10)
- No "Unknown" fallback
