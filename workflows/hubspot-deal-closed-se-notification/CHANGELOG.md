# Changelog

## v1.0 (2026-04-09)

Initial implementation.

- HubSpot internal workflow triggers webhook when deal enters 06_Sales Closed
- Normalize webhook payload to extract deal ID (same pattern as line-items workflow)
- Fetch deal details and check for SE assignment
- Resolve SE name via HubSpot Owners API
- Search Linear for existing ticket linked to deal (attachment-based dedup, same as SE-Linear workflow)
- Filter to Solutions Engineering team tickets only
- Build and post Slack message with deal link, SE name, Linear ticket link, and CTA
- Silent skip on all negative paths (no SE, no ticket, wrong stage)
- Connected to global error handler workflow
