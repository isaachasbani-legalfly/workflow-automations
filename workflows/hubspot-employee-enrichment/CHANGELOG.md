# Changelog

## v3.0 - 2026-03-05

Simplified workflow: removed scrape fallback, rewrote classification prompt for much lower subsidiary rate.

### Removed
- **Phase 5: Scrape Fallback** — 6 nodes deleted: Check Has Domain, Jina Scrape, Prep Estimate Prompt, Check Has Content, Gemini Estimate, Parse Estimate
- **"Extracted" enrichment source** — No longer scraping websites for employee counts
- **prompt-estimate-employees.md** — Scrape extraction prompt (deleted)
- **prompt-gemini.md** — Legacy v1.0 prompt (deleted)

### Changed
- **Classification prompt rewrite** — Much more conservative. Only 5 specific evidence categories (name + domain only, no LinkedIn): Fortune Global 500/Big 4 + geography, explicit words (Branch/Office/Division), famous acquisitions (100% certain only), subdomains, duplicate TLD entries. Explicitly excludes airlines, geography-in-brand-name, government, universities, non-profits, consumer email domains (icloud.com, me.com). Target: 5-10% subsidiary rate (was ~39%).
- **Removed LinkedIn URLs from prompt** — HubSpot LinkedIn data is often wrong (e.g., "Fingular" linked to Apple's LinkedIn). Sending LinkedIn URLs caused Gemini to hallucinate parent relationships. Only company name and domain are now sent.
- **Rewired routing** — Check Resolved FALSE now goes directly to Prepare Default (was Check Has Domain)
- **Prepare Default** — Updated `$('Check Has Domain')` reference to `$('Check Resolved')`
- **Slack summary** — 3 categories: Amplemarket, Parent enriched, Default (removed "Extracted"). Version label updated to v3.0.
- **Workflow renamed** — "HubSpot Employee Count Enrichment v3.0"

### Architecture
- 34 nodes + 3 sticky notes = 37 total (down from 40 + 3 = 43 in v2.1)
- In-place update of workflow `TxZMblqjvC86tHAu`; v2.1 version history preserved as rollback
- Gemini now only used for classification (not extraction)

---

## v2.1 - 2026-03-04

Stricter classification, extraction instead of estimation, parent company HubSpot links.

### Changed
- **Gemini model upgrade** — Classification upgraded from `gemini-2.5-flash` to `gemini-2.5-pro` for stricter subsidiary detection
- **Stricter classification prompt** — Added "NOT indicators" section: geography in name, regional descriptors, country-code TLDs alone do NOT indicate subsidiary status. Requires STRONG EVIDENCE.
- **Extraction replaces estimation** — Gemini no longer "estimates" employee counts. New prompt extracts ONLY explicitly stated counts from website content. If none found, defaults to 5.
- **New enrichment source value** — `Extracted` = explicit count found on website. `Estimated` now means default (5) only.
- **Content threshold** — `hasContent` check changed from `content.length > 50` to `content.length > 0`
- **Temperature changes** — Classification stays 0.2, extraction lowered from 0.3 to 0.1
- **Slack summary** — Updated to 4 categories: Amplemarket, Parent enriched, Extracted, Default

### New
- **Parent company HubSpot link** — 3 new nodes (Prep Parent Lookup, Search Parent HubSpot, Merge Parent Results). If parent company exists in HubSpot, `parent_company_name` is written as a clickable HubSpot URL; otherwise plain text name.

### Architecture
- 43 nodes (40 + 3 sticky notes), up from 40 in v2.0
- In-place update of workflow `TxZMblqjvC86tHAu`; v2.0 version history preserved as rollback

---

## v2.0 - 2026-02-26

Complete redesign to fix v1.0 limitations: branch detection after enrichment, no fallback, sequential API calls.

### New
- **Gemini batch classification BEFORE Amplemarket** — Identifies subsidiaries upfront so Amplemarket looks up the parent company (e.g., KPMG instead of KPMG Spain)
- **Batch Amplemarket enrichment** — Single POST + poll loop replaces up to 200 sequential GETs
- **Jina + Gemini scrape fallback** — Companies Amplemarket can't resolve get estimated from website content
- **Default = 5 employees** — Non-zero default for companies with no data source (tagged as "Estimated")
- **Paginated HubSpot fetch** — Cursor-based pagination (200/page) processes ALL yesterday's companies
- **New HubSpot properties**: `is_subsidiary` (checkbox), `parent_company_name` (text)
- **3-way routing** — (A) no count, (B) has count + subsidiary, (C) has count + independent (skip)
- **Preserve Pre-Update pattern** — Proven from industry-categorization workflow

### Changed
- Processes ALL companies created yesterday (not just those missing employee count)
- Enrichment source values: `Amplemarket`, `Amplemarket (parent: X)`, `Estimated` (replaces `Unresolved`)
- Slack channel changed to `C0AG86U9927`
- Schedule moved to 02:01 (was 02:00) to avoid overlap with v1.0

### Architecture
- 37 nodes + 3 sticky notes = 40 total (up from 17 in v1.0)
- Workflow ID: `TxZMblqjvC86tHAu` (new workflow, v1.0 preserved as rollback)

---

## v1.0 - 2026-02-25

Initial release.

- 17-node workflow: Schedule Trigger -> HubSpot fetch -> Amplemarket enrichment -> Gemini branch detection -> HubSpot update -> Slack summary
- Amplemarket lookup cascade: LinkedIn URL first, domain fallback
- Gemini 2.5 Flash for branch/subsidiary detection (temperature 0.1)
- Writes `numberofemployees` + `number-employees-enrichment-source` to HubSpot
- Daily Slack summary with enrichment statistics
- Error workflow: `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
- Retry config (3x/2s) on Amplemarket and Gemini API calls
- `continueRegularOutput` on Amplemarket nodes for graceful failure handling
