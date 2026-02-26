# Changelog

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
