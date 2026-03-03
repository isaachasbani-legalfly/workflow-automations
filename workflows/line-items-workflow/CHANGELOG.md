# Changelog

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
