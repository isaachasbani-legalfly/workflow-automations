# Changelog

## [2.1.0] - 2026-03-18

### Added
- **Product flag detection**: Line items containing "sso" (case-insensitive) set company property `has_sso` to "Yes"; line items containing "legal radar" set `has_legal_radar` to "true"
- **Pipeline-aware flag logic**: New Business deals set both flags explicitly (Yes/No or true/false). Expansion deals only set flags to Yes when found — never downgrade an existing Yes from a prior deal
- Detection uses case-insensitive substring match on raw line item names for resilience against naming variations
- Flags OR across multiple deals for the same company (if any deal has the product, the flag is set)
- No new nodes or API calls — flags are detected in Format Line Items, flow through Group by Company and Append & Prepare, and are included in the existing Update Company PATCH call

### Changed
- **Format Line Items**: Now outputs `hasSSO`, `hasLegalRadar`, and `isNewBusiness` boolean flags alongside existing `companyId` and `newLineItems`
- **Group by Company**: ORs `hasSSO`, `hasLegalRadar`, and `isNewBusiness` flags across multiple deals for the same company
- **Append & Prepare Update**: Passes through `hasSSO`, `hasLegalRadar`, and `isNewBusiness` from grouped data
- **Update Company**: PATCH body includes `has_sso` and `has_legal_radar` based on pipeline context — New Business sets both explicitly, Expansion only sets when product is found. Respects each property's HubSpot enumeration values (`has_sso`: "Yes"/"No", `has_legal_radar`: "true"/"false")

**Workflow ID**: `TQHGk5e2V0XL8D4f`

---

## [2.0.0] - 2026-03-03

### Changed
- **Production-only**: Removed manual trigger and backfill path (Search Closed Won Deals, Extract Deal IDs nodes)
- **Lifecycle filter removed**: All companies updated regardless of lifecycle stage — new clients signing their first deal are now captured
- **Formatting**: New Option A format with `[DD/MM/YYYY · Pipeline]` header per deal group
- **Deduplication**: Exact string matching prevents duplicate lines on webhook re-fire; blank lines (`""`) excluded from deduplication to preserve group separators
- **Append separator**: Uses `\n\n` between old and new content for consistent blank line between deal groups across batches
- **Skip instead of throw**: Deals with no line items or no associated company are silently skipped (no workflow crash)
- **Group by Company node added**: Merges multiple deal blocks for the same company with `\n\n`
- **Error handler linked**: Workflow errors now route to `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
- **Workflow activated**: Live on production webhook

**Workflow ID**: `TQHGk5e2V0XL8D4f`

---

## [1.0.0] - 2026-03-03

### Added
- Webhook trigger for production use (receives dealId from HubSpot internal workflow)
- Manual trigger for backfilling existing closed-won deals
- Search for closed-won deals in Sales Pipeline (`closedwon`) and Expansion Pipeline (`4914500817`)
- Single API call to get deal properties + associations (line items, companies)
- Batch read for line item details (name, net price, quantity, currency)
- Line item formatting: `Product Name - Qty X - €Price - Closed Won on DD/MM/YYYY`
- Currency symbol mapping for EUR, USD, GBP, CAD
- Append logic to preserve existing `productpurchased` values
- Retry on fail (3 attempts, 1s wait) for all HTTP Request nodes
- Sticky notes documenting production, backfill, and processing paths

**Workflow ID**: `TQHGk5e2V0XL8D4f`
