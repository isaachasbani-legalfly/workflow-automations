# Changelog

All notable changes to the HubSpot Company Industry Categorization workflow will be documented in this file.

---

## [3.3.3] - 2026-02-16

### Changed - No-description fallback: LinkedIn URL check replaces domain check

**Previous behavior** (no description):
```
No Description → Check Domain → YES: Amplemarket(domain) → LinkedIn data → ...
                              → NO: Manual Review
```

**New behavior** (no description):
```
No Description → Check LinkedIn URL → YES: Amplemarket(linkedin_url) → LinkedIn data → ...
                                    → NO: Website Scraping → ...
```

**Changes**:
1. `Check Has URL/Domain` → renamed to `Check Has LinkedIn URL`
   - Condition changed from `$json.domain` → `$json.linkedinUrl`
2. `Amplemarket LinkedIn API` body updated:
   - Old: `{ domain: $json.domain, company_name: $json.companyName }`
   - New: `{ linkedin_url: $json.linkedinUrl, company_name: $json.companyName }`
3. FALSE branch rewired: `Check Has LinkedIn URL` FALSE → `Website Scraping` (was `Send Manual Review Slack`)

**Why**: Using the HubSpot `linkedin_company_page` URL directly is more reliable than Amplemarket's domain-based LinkedIn lookup. When there's no LinkedIn URL, website scraping is attempted immediately — no dead-end manual review at this stage.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.3.2] - 2026-02-16

### Fixed - Slack notification showing "undefined" values and "Unknown" label

**Problem 1**: Slack messages displayed "undefined categorized as undefined" instead of company name and category.

**Root Cause**: The "Send Success Slack" node was reading `$json.companyName` and `$json.category` from its immediate input (the "Update HubSpot" node), which returns HubSpot's API response, not the categorization data.

**Fix 1**: Updated Slack node expressions to reference the "Check Confidence" node output, which contains the parsed categorization data:
- Changed: `$json.companyName` → `$('Check Confidence').item.json.companyName`
- Changed: `$json.category` → `$('Check Confidence').item.json.category`
- Changed: `$json.enrichmentSource` → `$('Check Confidence').item.json.enrichmentSource`
- Changed: `$json.companyId` → `$('Check Confidence').item.json.companyId`

Also fixed expression syntax from `{{ }}` (Mustache) to `={{ }}` (n8n expressions) using template literals.

**Problem 2**: Slack displayed "Unknown" instead of user-friendly "Others" label.

**Root Cause**: Parse Gemini Response nodes map "Others" → "Unknown" for HubSpot's internal enum, but Slack was showing the internal value.

**Fix 2**: Added inline mapping in Slack message to display "Others" instead of "Unknown":
```javascript
${$('Check Confidence').item.json.category === 'Unknown' ? 'Others' : $('Check Confidence').item.json.category}
```

**Result**: HubSpot stores "Unknown" (correct enum), Slack displays "Others" (user-friendly label).

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.3.1] - 2026-02-16

### 🎯 Enhanced HubSpot Description Prompt - Comprehensive Classification Rules

**Changed - HubSpot Description Path (Path 1)**
- **New comprehensive prompt** with 9-rule classification system (3,763 characters)
- **Removed token limit** (`maxOutputTokens` removed - no truncation)
- **Removed MANUAL_REVIEW_REQUIRED** - always returns one of 16 categories
- **Model**: gemini-2.5-flash (unchanged)

**New Classification Rules**:
1. **Technology Priority**: Software/SaaS builders → Technology (even if serving specific industry)
2. **Legal Services Priority**: ALL legal work → Legal Services (never Consulting)
3. **Specific Service Categories**: HR consulting → HR and Payroll Services (not Consulting)
4. **Construction vs Manufacturing**: Buildings/infrastructure = Construction, products = Manufacturing
5. **Retail vs Manufacturing**: Companies that sell → Retail (even if they manufacture)
6. **Public Sector**: Only government agencies (contractors → their service category)
7. **Multi-Service**: Multiple services → Consulting (default for generalist firms)
8. **Others**: Only for non-matching industries (Agriculture, Education, Media, Non-profits)
9. **Conflict Resolution**: Clear hierarchy (Technology > Legal Services > Specific Service > Consulting)

**Examples in Prompt**:
- "We build SaaS software for law firms" → Technology
- "We provide HR consulting and recruiting services" → HR and Payroll Services
- "Law firm specializing in corporate law" → Legal Services
- "We manufacture and sell consumer electronics online" → Retail and Consumer Goods
- "Multi-service firm offering accounting, HR, and legal advisory" → Consulting

**Technical Changes**:
- Prepare Gemini Input - HubSpot: 3,763 char prompt (was ~800 chars)
- Gemini Categorization - HubSpot: No maxOutputTokens (was 50)
- Parse Response - HubSpot: `needsManualReview: false` (always high confidence)

---

## [3.3.0] - 2026-02-16

### 🎯 Major Architecture Change: Multi-Path LLM System

**Changed**
- **THREE independent LLM paths** (previously: single merged path)
  - Path 1: HubSpot Description → Gemini (working - now ENHANCED with new prompt)
  - Path 2: LinkedIn Enrichment → Gemini (NEW)
  - Path 3: Website Scraping → Gemini (NEW)

**Added**
- 9 new nodes for independent categorization paths:
  - `Check Has URL/Domain` - IF node to verify domain exists
  - `Amplemarket LinkedIn API` - HTTP Request for LinkedIn enrichment
  - `Check LinkedIn Data Retrieved` - IF node to verify LinkedIn response
  - `Prepare Gemini Input - LinkedIn` - Code node with LinkedIn-optimized prompt
  - `Gemini Categorization - LinkedIn` - HTTP Request to Gemini API
  - `Parse Gemini Response - LinkedIn` - Code node to extract category
  - `Check Website Data Retrieved` - IF node to verify website scraping
  - `Prepare Gemini Input - Website` - Code node with website-optimized prompt
  - `Gemini Categorization - Website` - HTTP Request to Gemini API
  - `Parse Gemini Response - Website` - Code node to extract category

**Removed**
- `Merge Enrichment Branches` node (no longer needed with independent paths)

**Modified**
- Renamed existing nodes for clarity:
  - `Prepare Gemini Input` → `Prepare Gemini Input - HubSpot`
  - `Gemini Categorization` → `Gemini Categorization - HubSpot`
  - `Parse Gemini Response` → `Parse Gemini Response - HubSpot`
- Updated all connections to support three-path architecture

**Benefits**
- Data-source-specific prompts optimized for each enrichment type
- Independent processing eliminates data format conflicts
- HubSpot description path unchanged (proven working)
- Graceful fallback cascade maintained: LinkedIn → Website → Online Research
- All paths converge at "Check Confidence" for unified updates

**Technical Details**
- Total nodes: 27 (was 18, +9 new nodes)
- Node reuse: 5 shared nodes (Check Confidence, Update HubSpot, both Slack nodes)
- Path-specific: 9 new nodes (3 per path: Prepare, Gemini, Parse)

**Requirements**
- ⚠️ **Amplemarket API Key** required for LinkedIn enrichment path
- All other credentials unchanged (HubSpot, Slack, Gemini)

---

## [3.2.14] - 2026-02-16

### Fixed - Multi-branch convergence: 36 items dropped, only 3 (last batch) processed

**Root cause**: In n8n v1 execution order, when multiple separate branches converge on the same non-Merge node, only the **last arriving batch** propagates to downstream nodes. Earlier batches are silently discarded.

The workflow had 4 branches all connecting directly to `Prepare Gemini Input`:
- `Check Description Exists TRUE` → Prepare Gemini Input (36 items — **discarded**)
- `Check LinkedIn Success TRUE` → Prepare Gemini Input (some items — **discarded**)
- `Check Website Success TRUE` → Prepare Gemini Input (some items — **discarded**)
- `Online Research` → Prepare Gemini Input (3 items — **only these survived**)

This is why the execution showed 36 inputs → 3 outputs (all OVO Energy from Online Research).

**Fix**: Added a `Merge Enrichment Branches` node (mode: Append) between all 4 branches and `Prepare Gemini Input`. All 4 branches now feed into the Merge node, which collects the full batch before passing to Prepare Gemini Input.

**Also fixed**: `Parse Gemini Response` had the same `$node['X'].json` bug (always returns item[0]). Changed to `$('Prepare Gemini Input').item.json` to correctly pair each Gemini response with its source company data.

**Expected behaviour after fix**: 39 companies processed → 39 items into Prepare Gemini Input → 39 Gemini calls → 39 HubSpot updates + Slack notifications.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.13] - 2026-02-16

### Fixed - All companies classified as OVO Energy ($node vs $() reference bug)

**Root cause**: `Prepare Gemini Input` used `$node['Prepare Company Data'].json` to get company data. In n8n, `$node['NodeName'].json` ALWAYS returns the **first item** of that node's output — regardless of which item is currently being processed. Since OVO Energy was the first company in the batch, every single company's Gemini prompt was built with OVO Energy's name, description, and domain, and every HubSpot update wrote to OVO Energy's companyId.

**Confirmed via HubSpot**: Only OVO Energy had `industry__internal_` set from execution #105, despite 44 companies being processed.

**Fix**: Changed `$node['Prepare Company Data'].json` → `$('Prepare Company Data').item.json`
- `$('NodeName').item.json` follows the item's execution lineage and returns the **correct corresponding item** from that node, regardless of which branch the item arrived from (direct description path, LinkedIn, website, or Online Research).

**Why OVO Energy appeared 3 times**: 3 duplicate OVO Energy company records exist in HubSpot created today without descriptions. These 3 records took the longest path (no description → LinkedIn failed → website failed → Online Research) and emerged as the last batch in the execution — which is why the execution trace shows 3 items.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.9] - 2026-02-16

### Fixed - Gemini model deprecated (gemini-1.5-flash → gemini-2.0-flash)

**Error**: `404 - models/gemini-1.5-flash is not found for API version v1beta`

**Root cause**: `gemini-1.5-flash` has been deprecated and removed from the v1beta endpoint.

**Fix**: Updated Gemini Categorization URL to use `gemini-2.0-flash`:
```
https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent
```

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.8] - 2026-02-16

### Fixed - "Others" label vs "Unknown" internal enum mismatch

**Problem**: HubSpot `industry__internal_` property stores `"Unknown"` internally but displays it as `"Others"` in the UI. Gemini was being instructed to respond with `"Unknown"` (the internal value), which is confusing and inconsistent with what users see.

**Fix**:
- Updated `Prepare Gemini Input` prompt: 16th category now reads `"Others"` (matching HubSpot UI label)
- Updated `Parse Gemini Response`: added mapping `if (category === 'Others') category = 'Unknown'` to convert Gemini's label back to the HubSpot enum value before the property update

**Result**: Gemini works with human-readable labels; HubSpot receives the correct internal value.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.7] - 2026-02-16

### Fixed - Replace LangChain Gemini sub-node with direct REST API call

**Root cause**: `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` is a LangChain **sub-node** designed to be wired as an AI connection into agents/chains — not for direct execution in the main flow. When called directly it makes a malformed request to Google and returns a 400 HTML error page.

**Fix 1**: Replaced `Gemini Categorization` node:
- Old: `@n8n/n8n-nodes-langchain.lmChatGoogleGemini` (typeVersion 1)
- New: `n8n-nodes-base.httpRequest` (typeVersion 4.2) calling `POST https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent`
- Authentication: `predefinedCredentialType: "googlePalmApi"` (same credential)
- Body: `JSON.stringify({ contents: [{ parts: [{ text: $json.prompt }] }], generationConfig: { temperature: 0.3, maxOutputTokens: 50 } })`

**Fix 2**: Updated `Parse Gemini Response` to read from REST API response structure:
- Old: `$json.text || $json.response` (LangChain output format)
- New: `response?.candidates?.[0]?.content?.parts?.[0]?.text` (Gemini REST API format)

**Fix 3**: Corrected HubSpot enum value in `Prepare Gemini Input` prompt:
- Old: `"Payroll and HR Services"` (invalid enum — not accepted by HubSpot)
- New: `"HR and Payroll Services"` (exact HubSpot enum value)
- Also: expanded from 15 → 16 categories by adding `Unknown`

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.6] - 2026-02-16

### Fixed - HubSpot v2 API nested property format & Gemini source detection

**Two root causes found from execution #98:**

**1. `companyId: null` + `companyName: {value: "OVO Energy", ...}` (object, not string)**
- `Get Company Details` HubSpot node uses the v2 API internally, which returns:
  - `companyId` as the ID field (not `id`)
  - Properties as deeply nested objects: `{value: "...", timestamp: ..., source: ..., versions: [...]}`
- `Prepare Company Data` was reading `$json.id` (null) and `$json.properties.name` (the full object)
- Fix: Updated all 6 assignments to handle both v2 (object) and v3 (string) formats:
  - `companyId`: `$json.companyId || $json.id`
  - Properties: `typeof val === 'object' ? val.value : val`

**2. Gemini always received "Minimal data available" (enrichmentSource always "fallback")**
- `Prepare Gemini Input` code tried `allItems[0]?.node` to detect which branch sent the item
- `node` is not a property on n8n items → always empty string → always falls to "fallback"
- Fix: Detect source by inspecting `$json` structure:
  - `$json.organic_results` → online_research
  - `$json.data` (HTML string) → website
  - `$json.description` ≠ company description → linkedin
  - Otherwise → hubspot (description was available directly)

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.5] - 2026-02-16

### Fixed - Check Demo Form routing was inverted

**Problem**: Companies that submitted a form with their industry (non-null `industry__form____contact_sync`) were being sent through the full categorization pipeline. Companies without form data were being skipped.

**Correct logic**: If a company filled a form with their industry, that value is trusted — skip AI categorization. Only companies with no form data should go through Gemini categorization.

**Fix**: Changed `notEmpty` → `empty` operator on the IF node:
- TRUE (form IS empty = no industry data) → Get Company Details → categorize ✓
- FALSE (form has data = industry known) → skip ✓

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.4] - 2026-02-16

### Fixed - jsonBody must return a JSON string, not a JS object

**Error**: `NodeOperationError: JSON parameter needs to be valid JSON`

**Root Cause**: The `jsonBody` field type is "json". n8n evaluates the expression `={{ { ... } }}` to a JavaScript object, then the HTTP Request node calls `JSON.parse(object)`. Since `JSON.parse` coerces non-strings via `.toString()`, this becomes `JSON.parse("[object Object]")` → throws.

**Fix**: Wrapped the object in `JSON.stringify(...)` so the expression returns a valid JSON string:
```
={{ JSON.stringify({ filterGroups: [...], ... }) }}
```
Now `JSON.parse(string)` succeeds inside the node.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.3] - 2026-02-16

### Fixed - HTTP Request missing specifyBody parameter

**Problem**: "Search Today's Companies" was still returning all 37,242 companies despite the `jsonBody` fix in v3.2.2.

**Root Cause**: `specifyBody` defaults to `"keypair"` (key-value pairs mode). Without explicitly setting `specifyBody: "json"`, n8n ignores `jsonBody` entirely and sends an empty body to HubSpot. An empty POST body to `/crm/v3/objects/companies/search` returns all companies with no filtering.

**Fix**: Added `specifyBody: "json"` to the "Search Today's Companies" HTTP Request node parameters. The full required config for JSON body in HTTP Request v4.2 is:
```
sendBody: true
contentType: "json"        ← sets Content-Type: application/json
specifyBody: "json"        ← tells n8n to use jsonBody field  ← THIS was missing
jsonBody: "={{ { ... } }}" ← the actual body content
```

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.2] - 2026-02-16

### Fixed - HTTP Request body parameter name & Code node mode

**Problem 1**: "Search Today's Companies" was returning ALL 37,242 companies (no date filter applied).
- Root cause: `body` parameter name is wrong for HTTP Request v4.2 with `contentType: "json"` — the filter was silently ignored
- Fix: Renamed `body` → `jsonBody` (the correct parameter name for JSON body in typeVersion 4.2)

**Problem 2**: "Split Companies" Code node error: `"A 'json' property isn't an object [item 0]"`
- Root cause: `mode: "runOnceForEachItem"` receives one item at a time, but the node was returning an array where each array element is an item — n8n interprets the array as the single output item's `json`, which is an array (not an object)
- Fix: Changed `mode` to `"runOnceForAllItems"`, updated code to use `$input.first().json.results` to access the HTTP response and map each company to a flat `{json: {id, properties}}` item

**Flow** (unchanged):
```
Schedule Trigger → Search Today's Companies (HTTP POST w/ createdate GTE filter) → Split Companies (one item per company) → Check Demo Form → ...
```

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.1] - 2026-02-16

### Fixed - $helpers not available in n8n task runner

**Error**: `ReferenceError: $helpers is not defined` — n8n's external task runner does not expose `$helpers` to Code nodes.

**Fix**: Replaced the single "Get Today's Companies" Code node with two nodes:
1. **HTTP Request "Search Today's Companies"** — calls `POST /crm/v3/objects/companies/search` with `createdate >= today_midnight_UTC` filter using the `hubspotAppToken` predefined credential
2. **Code "Split Companies"** — splits `$json.results` array into individual `{id, properties}` items

**Flow**:
```
Schedule Trigger → Search Today's Companies (HTTP Request) → Split Companies (Code) → Check Demo Form
```

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.2.0] - 2026-02-16

### Refactor - Replace polling node with HubSpot Search API (efficient date filtering)

**Problem**: `getRecentlyCreatedUpdated` with `returnAll: true` retrieves every company ever created and filters in n8n — extremely inefficient.

**Solution**: Replaced the HubSpot poll node + Normalize Code node with a single "Get Today's Companies" Code node that:
- Calls `POST /crm/v3/objects/companies/search` with `createdate >= today_midnight_UTC`
- Filters happen at the HubSpot API level — only today's companies are returned
- Handles pagination automatically (loops through pages of 100)
- Returns `{id, properties}` in v3 flat format directly — no normalization needed

**Flow change**:
```
Before: Schedule Trigger → Get Recent Companies (HubSpot) → Normalize & Filter New (Code) → Check Demo Form
After:  Schedule Trigger → Get Today's Companies (Code)                                   → Check Demo Form
```

**Nodes removed**: Get Recent Companies (HubSpot `getRecentlyCreatedUpdated`)
**Nodes merged**: "Normalize & Filter New" replaced by "Get Today's Companies"

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.1.6] - 2026-02-16

### Fixed - Use returnAll to guarantee all today's companies are fetched

**Problem**: `limit: 100` would miss companies if more than 100 were added today.

**Fix**: Changed to `returnAll: true` — HubSpot node paginates through all recently created companies; the Normalize node then filters down to only those with `createdate` = today.

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## [3.1.5] - 2026-02-16

### Changed - Fetch All Companies Added Today (not just uncategorized)

**Previous behavior**: Only fetched companies missing `industry__internal_` (uncategorized)
**New behavior**: Fetches all companies added today — including already-categorized ones — so existing categories can be improved/corrected

**Changes**:
- Removed `industry__internal_` `NOT_HAS_PROPERTY` filter from Get Recent Companies node
- Increased limit from 10 → 100 to handle larger daily volumes
- Renamed node from "Get Uncategorized Companies" → "Get Recent Companies"
- Normalize node still filters to today's `createdate` only

**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

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
