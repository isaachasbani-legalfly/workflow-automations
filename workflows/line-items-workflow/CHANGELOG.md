# Changelog

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
