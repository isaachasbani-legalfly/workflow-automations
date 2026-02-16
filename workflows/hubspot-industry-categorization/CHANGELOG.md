# Changelog

All notable changes to the HubSpot Company Industry Categorization workflow will be documented in this file.

## [3.1.4] - 2026-02-16

### Fixed - Three Additional Issues

**1. Get Uncategorized Companies filter lost**
- Partial update to the Normalize node inadvertently stripped the `industry__internal_` `NOT_HAS_PROPERTY` filter
- Restored: workflow now only fetches companies without `industry__internal_` set

**2. Check Demo Form leftValue simplified**
- Removed the `typeof` ternary expression (unnecessary after Normalize guarantees strings)
- New expression: `={{ $json.properties.industry__form____contact_sync || '' }}`

**3. Check Website Success leftValue wrong type**
- Was `={{ $json }}` — passes entire JSON object to string `contains` operator → "Wrong type: [object Object]" error
- Fixed to `={{ $json.data || '' }}` — accesses the HTML body string from the HTTP response

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.1.3] - 2026-02-16

### Fixed - IF Node Object Type Error & Time Filter Change

**Issues**:
1. `Check Demo Form` still throwing "Wrong type: '[object Object]' is an object but was expecting a string" — the previous normalization only handled `{value: "..."}` format, leaving other object shapes as-is
2. Time filter was "last 25 hours" instead of "companies added today"

**Fixes** (both in "Normalize & Filter New" Code node):
- **Stronger normalization**: ALL property values are now converted to strings. Any `null`/`undefined` → `''`, any object → `String(val.value)` or `''`, primitives → `String(val)`. This guarantees downstream IF nodes always receive strings.
- **Today filter**: Changed from `Date.now() - 25h` rolling window to `Date.UTC` midnight of current day, so companies added at any point today are included regardless of when the workflow runs.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.1.2] - 2026-02-16

### Fixed - HubSpot v2 API companyId Mismatch & Retroactive Processing

**Root Cause**: `getRecentlyCreatedUpdated` uses the legacy HubSpot v2 API which:
- Returns `companyId` (not `id`) as the identifier field
- Returns properties in nested `{value: "..."}` format instead of flat strings
- This caused `$json.id` to be `undefined`, triggering 400 errors in "Get Company Details"

**Additional Fix**: Workflow previously had no time filter, meaning it would retroactively process all 3,511 existing uncategorized companies. Added a 25-hour rolling window so only companies created since the last run are processed.

**Fix Applied**: Added "Normalize & Filter New" Code node between "Get Uncategorized Companies" and "Check Demo Form":
- Remaps `companyId` → `id` (fixes v2 API field name mismatch)
- Filters out companies older than 25 hours (prevents retroactive processing)
- Normalizes nested v2 property format `{value: "..."}` to flat strings

**Also Fixed in this session**:
- `Check Demo Form` IF node: added `typeof` check to handle object-type property values (prevents "Wrong type: '[object Object]'" error)
- HubSpot property name corrected: `industry_internal_sync` → `industry__internal_` (actual property name in HubSpot)

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.1.1] - 2026-02-16

### Fixed - HTTP Request Node "Not Installed" Error

**Root Cause**: The three HTTP Request nodes (`LinkedIn Enrichment`, `Website Scraping`, `Online Research`) were deployed with `typeVersion: 4.4`, which is not supported by the n8n cloud instance. This caused:
- UI: "Install this node to use it" / "This node is not currently installed"
- Runtime: `Cannot read properties of undefined (reading 'execute')`

**Fix Applied (via n8n MCP)**:
- Downgraded all three HTTP Request nodes from `typeVersion: 4.4` → `typeVersion: 4.2`
- Cleaned up `LinkedIn Enrichment` body params: replaced `specifyBody: "json"` with `contentType: "json"` and `body` expression
- Updated `Website Scraping` and `Online Research` expressions to use `$('Node Name').item.json` syntax

**Nodes Fixed**:
| Node | Change |
|------|--------|
| LinkedIn Enrichment | typeVersion 4.4 → 4.2, specifyBody → contentType |
| Website Scraping | typeVersion 4.4 → 4.2 |
| Online Research | typeVersion 4.4 → 4.2 |

**Workflow ID**: `8DM3CwXLxOT3G8B7`
**Deployed**: https://legalfly.app.n8n.cloud/workflow/8DM3CwXLxOT3G8B7

---

## [1.0.0] - 2026-02-15

### Initial Release
- **Feature**: Automatic company industry categorization using Google Gemini 3 Flash AI
- **Trigger**: HubSpot company.creation webhook event
- **Enrichment Strategy**:
  - Primary: Amplemarket API for LinkedIn company data (description, industry, specialties)
  - Fallback: HTTP website metadata extraction (Open Graph tags, meta descriptions)
  - Error handling: Slack alert if both enrichment methods fail
- **Demo Form Detection**: Skips categorization for companies from demo form (`industry__form____contact_sync` field)
- **Classification**: 15 internal industry categories with rules-based logic
- **Confidence Threshold**: 70% - categorizations below this are flagged for manual review
- **Notification System**:
  - Success: Slack message with category and HubSpot link
  - Manual Review: Slack alert with available data for team review
  - Enrichment Failed: Slack alert when both data sources unavailable
- **HubSpot Integration**: Updates `industry_internal_sync` property on successful categorization

### Node Configuration
- **HubSpot Trigger**: Listens for company creation events
- **Check Demo Form**: IF node to skip demo form submissions
- **Get Company Details**: Fetches full company record from HubSpot
- **Prepare LinkedIn Call**: Extracts relevant company data for enrichment
- **Amplemarket LinkedIn Enrichment**: HTTP POST to LinkedIn company search endpoint
- **LinkedIn Success Check**: IF node to validate enrichment response
- **Extract LinkedIn Data**: Parses LinkedIn company data
- **Website Metadata Fetch**: HTTP GET to extract website metadata
- **Website Success Check**: IF node to validate website response
- **Extract Website Data**: Parses HTML meta tags for description
- **Both Failed - Error**: Sets error flag if both enrichment sources fail
- **Merge Enriched Data**: Combines HubSpot native fields with enriched data
- **Gemini Categorization**: LangChain Google Gemini Chat Model with structured prompt
- **Check Manual Review**: IF node to route by confidence level
- **Parse Categorization Result**: Extracts category from Gemini response
- **Update HubSpot with Category**: Updates company property in HubSpot
- **Send Success Slack**: Notifies team of successful categorization
- **Send Manual Review Slack**: Alerts for uncertain categorizations
- **Send Enrichment Failed Alert**: Alerts when enrichment data unavailable

### Data Flow
```
Company Created → Demo Form Check → Get Details → LinkedIn Enrichment → Website Fallback
→ Data Merge → Gemini Categorization → Confidence Check → HubSpot Update + Slack Notification
```

### Validation Status
✅ Workflow validation passed
- 19 nodes configured correctly
- All connections valid
- All expressions validated
- Ready for deployment to n8n instance

### Next Steps
1. Create test data with sample companies
2. Deploy to n8n instance at https://legalfly.app.n8n.cloud
3. Monitor Slack channel #n8n-helper-test-bot for real-time notifications
4. Collect feedback on categorization accuracy
5. Plan iterative improvements based on manual review patterns
